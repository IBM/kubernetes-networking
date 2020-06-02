# Lab05 - Kubernetes Network Policy and Calico 

NetworkPolicy resources use labels to select pods and define rules which specify what traffic is allowed to the selected pods. By default, pods accept traffic from any source. Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections not allowed by any NetworkPolicy. 

Network policies do not conflict; they are additive. If any policy or policies select a pod, the pod is restricted to what is allowed by the union of those policies’ ingress/egress rules. Thus, order of evaluation does not affect the policy result.

There are four kinds of selectors in an ingress `from` section or egress `to` section:
- podSelector,
- namespaceSelector,
- podSelector and namespaceSelector,
- ipBlock for IP CIDR ranges.

For example,

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - protocol: TCP
      port: 5978
```

Every IBM Cloud Kubernetes Service cluster is set up with a network plug-in called Calico. Default network policies are set up to secure the public network interface of every worker node in the cluster.

You can use Kubernetes and Calico to create network policies for a cluster. With Kubernetes network policies, you can specify the network traffic that you want to allow or block to and from a pod within a cluster. To set more advanced network policies such as blocking inbound (ingress) traffic to network load balancer (NLB) services, use Calico network policies.

When a Kubernetes network policy is applied, it is automatically converted into a Calico network policy so that Calico can apply it as an Iptables rule. Both incoming and outgoing network traffic can be allowed or blocked based on protocol, port, and source or destination IP addresses. Traffic can also be filtered based on pod and namespace labels.

Calico network policies are a superset of the Kubernetes network policies and are applied by using calicoctl commands. Calico policies add the following features:
- Allow or block network traffic on specific network interfaces regardless of the Kubernetes pod source or destination IP address or CIDR.
- Allow or block network traffic for pods across namespaces.
- Block inbound traffic to Kubernetes LoadBalancer or NodePort services.

Calico enforces these policies, including any Kubernetes network policies that are automatically converted to Calico policies, by setting up Linux Iptables rules on the Kubernetes worker nodes. Iptables rules serve as a firewall for the worker node to define the characteristics that the network traffic must meet to be forwarded to the targeted resource.

# Adopt a zero trust network model

Adopting a zero trust network model is best practice for securing workloads and hosts in your cloud-native strategy.

## Create helloworld Proxy

Deploy the `helloworld` proxy,

```
$ kubectl create -f helloworld-proxy-deployment.yaml
$ kubectl create -f helloworld-proxy-service-loadbalancer.yaml
```

Get the proxy service details and test the proxy,

```
$ kubectl get svc helloworld-proxy
NAME    TYPE    CLUSTER-IP    EXTERNAL-IP    PORT(S)    AGE
helloworld-proxy    LoadBalancer    172.21.78.158    169.48.67.166    8080:32333/TCP    61s
```

To test use the External-IP and the NodePort, e.g. 169.48.67.166:32333, and send the proxy the Kubernetes service name and target port,

```
$ curl -L -X POST "http://169.48.67.166:32333/proxy/api/messages" -H 'Content-Type: application/json' -H 'Content-Type: application/json' -d '{ "sender": "remko", "host": "helloworld-proxy:8080" }'

{"id":"3aa4f889-94a0-4be8-b8f2-8b59f8ad3de7","sender":"remko","message":"Hello remko (proxy)","host":"helloworld-proxy:8080"}
```


## Apply Network Policy - Allow No Traffic

Define the Network Policy file,

```
$ echo 'apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: helloworld-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress' > helloworld-policy-denyall.yaml
```

Create the Network Policy,

```
$ kubectl create -f helloworld-policy-denyall.yaml
networkpolicy.networking.k8s.io/helloworld-deny-all created
```

Test the `helloworld` and the proxy,
```
$ curl -L -X POST "http://169.48.67.163:31777/api/messages" -H 'Content-Type: application/json' -d '{ "sender": "remko" }'
curl: (7) Failed to connect to 169.48.67.163 port 31777: Connection timed out

$ curl -L -X POST "http://169.48.67.166:32333/proxy/api/messages" -H 'Content-Type: application/json' -H 'Content-Type: application/json' -d '{ "sender": "remko", "host": "helloworld:8080" }'
curl: (7) Failed to connect to 169.48.67.166 port 32333: Connection timed out
```

It takes quite a long time before connections time out. All traffic is denied, despite that we have a LoadBalancer service added to each deployment,

```
$ kubectl get svc
NAME    TYPE    CLUSTER-IP    EXTERNAL-IP    PORT(S)    AGE
helloworld    LoadBalancer    172.21.161.255    169.48.67.163    8080:31777/TCP    3h31m
helloworld-proxy    LoadBalancer    172.21.244.220    169.48.67.166    8080:32333/TCP    11m
kubernetes    ClusterIP    172.21.0.1    <none>    443/TCP    8h
```


## Apply Network Policy - Allow Only Traffic From Proxy

Let's allow traffic to the proxy. 

Define the Network Policy file,

```
$ echo 'apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: helloworld-allow-test
spec:
  selector: app == 'helloworld'
  types:
  - Ingress
  - Engress
  ingress:
  - action: Allow
    protocol: TCP
    destination: 
      ports:
      - 8080' > helloworld-calico-allow-test.yaml
```

Create the Network Policy,

```
$ kubectl create -f helloworld-policy-deny-to-label.yaml
networkpolicy.networking.k8s.io/helloworld-allow-to-label created
```

Test the `helloworld` and the proxy,
```
$ curl -L -X POST "http://169.48.67.163:31777/api/messages" -H 'Content-Type: application/json' -d '{ "sender": "remko" }'

curl: (7) Failed to connect to 169.48.67.163 port 31777: Connection timed out

$ curl -L -X POST "http://169.48.67.166:32333/proxy/api/messages" -H 'Content-Type: application/json' -d '{ "sender": "remko", "host": "helloworld:8080" }'

curl: (7) Failed to connect to 169.48.67.166 port 32333: Connection timed out
```


```
kubectl get networkpolicy
kubectl delete networkpolicy helloworld-allow-to-label
```


$ wget https://github.com/projectcalico/calicoctl/releases/download/v3.14.1/calicoctl
$ chmod 755 calicoctl
$ echo 'export PATH=$HOME:$PATH' > .bash_profile
$ source .bash_profile
$ ibmcloud ks cluster config --cluster remkohdev-iks116-2n-cluster --admin --network

/tmp/ic/cloudshell-9aecc719-0b60-456b-b53f-b2dd029757da-1-5f456cb5nnwgc-1/.bluemix/plugins/container-service/clusters/remkohdev-iks116-2n-cluster-br9v078d0qi43m0e31n0-admin/calicoctl.cfg

$ mv /tmp/ic/cloudshell-9aecc719-0b60-456b-b53f-b2dd029757da-1-5f456cb5nnwgc-1/.bluemix/plugins/container-service/clusters/remkohdev-iks116-2n-cluster-br9v078d0qi43m0e31n0-admin/calicoctl.cfg /etc/calico

$ calicoctl get nodes
NAME                                                     
kube-br9v078d0qi43m0e31n0-remkohdevik-default-0000014b   
kube-br9v078d0qi43m0e31n0-remkohdevik-default-000002e0

$ calicoctl version
Client Version:    v3.10.0
Git commit:        7968b525
Cluster Version:   v3.9.5
Cluster Type:      k8s,bgp

View network policy

$ calicoctl get hostendpoint -o yaml
apiVersion: projectcalico.org/v3
items:
- apiVersion: projectcalico.org/v3
  kind: HostEndpoint
  metadata:
    creationTimestamp: 2020-05-31T18:23:51Z
    labels:
      arch: amd64
      beta.kubernetes.io/arch: amd64
      beta.kubernetes.io/instance-type: b3c.4x16.encrypted
      beta.kubernetes.io/os: linux
      failure-domain.beta.kubernetes.io/region: us-south
      failure-domain.beta.kubernetes.io/zone: dal13
      ibm-cloud.kubernetes.io/encrypted-docker-data: "true"
      ibm-cloud.kubernetes.io/external-ip: 150.238.93.101
      ibm-cloud.kubernetes.io/ha-worker: "true"
      ibm-cloud.kubernetes.io/iaas-provider: softlayer
      ibm-cloud.kubernetes.io/internal-ip: 10.187.222.149
      ibm-cloud.kubernetes.io/machine-type: b3c.4x16.encrypted
      ibm-cloud.kubernetes.io/os: UBUNTU_18_64
      ibm-cloud.kubernetes.io/region: us-south
      ibm-cloud.kubernetes.io/sgx-enabled: "false"
      ibm-cloud.kubernetes.io/worker-id: kube-br9v078d0qi43m0e31n0-remkohdevik-default-0000014b
      ibm-cloud.kubernetes.io/worker-pool-id: br9v078d0qi43m0e31n0-3108b12
      ibm-cloud.kubernetes.io/worker-pool-name: default
      ibm-cloud.kubernetes.io/worker-version: 1.16.10_1533
      ibm-cloud.kubernetes.io/zone: dal13
      ibm.role: worker_private
      kubernetes.io/arch: amd64
      kubernetes.io/hostname: 10.187.222.149
      kubernetes.io/os: linux
      privateVLAN: "2847992"
      publicVLAN: "2847990"
    name: kube-br9v078d0qi43m0e31n0-remkohdevik-default-0000014b-worker-private
    resourceVersion: "2643"
    uid: e0eaf0d6-a36b-11ea-8f29-06ce448b0c2d
  spec:
    expectedIPs:
    - 10.187.222.149
    interfaceName: eth0
    node: kube-br9v078d0qi43m0e31n0-remkohdevik-default-0000014b
- apiVersion: projectcalico.org/v3
  kind: HostEndpoint
  metadata:
    creationTimestamp: 2020-05-31T18:23:39Z
    labels:
      arch: amd64
      beta.kubernetes.io/arch: amd64
      beta.kubernetes.io/instance-type: b3c.4x16.encrypted
      beta.kubernetes.io/os: linux
      failure-domain.beta.kubernetes.io/region: us-south
      failure-domain.beta.kubernetes.io/zone: dal13
      ibm-cloud.kubernetes.io/encrypted-docker-data: "true"
      ibm-cloud.kubernetes.io/external-ip: 150.238.93.101
      ibm-cloud.kubernetes.io/ha-worker: "true"
      ibm-cloud.kubernetes.io/iaas-provider: softlayer
      ibm-cloud.kubernetes.io/internal-ip: 10.187.222.149
      ibm-cloud.kubernetes.io/machine-type: b3c.4x16.encrypted
      ibm-cloud.kubernetes.io/os: UBUNTU_18_64
      ibm-cloud.kubernetes.io/region: us-south
      ibm-cloud.kubernetes.io/sgx-enabled: "false"
      ibm-cloud.kubernetes.io/worker-id: kube-br9v078d0qi43m0e31n0-remkohdevik-default-0000014b
      ibm-cloud.kubernetes.io/worker-pool-id: br9v078d0qi43m0e31n0-3108b12
      ibm-cloud.kubernetes.io/worker-pool-name: default
      ibm-cloud.kubernetes.io/worker-version: 1.16.10_1533
      ibm-cloud.kubernetes.io/zone: dal13
      ibm.role: worker_public
      kubernetes.io/arch: amd64
      kubernetes.io/hostname: 10.187.222.149
      kubernetes.io/os: linux
      privateVLAN: "2847992"
      publicVLAN: "2847990"
    name: kube-br9v078d0qi43m0e31n0-remkohdevik-default-0000014b-worker-public
    resourceVersion: "2642"
    uid: 78ec7195-2ddd-4b5f-b7c4-8a54d25f4beb
  spec:
    expectedIPs:
    - 150.238.93.101
    interfaceName: eth1
    node: kube-br9v078d0qi43m0e31n0-remkohdevik-default-0000014b
- apiVersion: projectcalico.org/v3
  kind: HostEndpoint
  metadata:
    creationTimestamp: 2020-05-31T18:23:01Z
    labels:
      arch: amd64
      beta.kubernetes.io/arch: amd64
      beta.kubernetes.io/instance-type: b3c.4x16.encrypted
      beta.kubernetes.io/os: linux
      failure-domain.beta.kubernetes.io/region: us-south
      failure-domain.beta.kubernetes.io/zone: dal13
      ibm-cloud.kubernetes.io/encrypted-docker-data: "true"
      ibm-cloud.kubernetes.io/external-ip: 150.238.93.100
      ibm-cloud.kubernetes.io/ha-worker: "true"
      ibm-cloud.kubernetes.io/iaas-provider: softlayer
      ibm-cloud.kubernetes.io/internal-ip: 10.187.222.146
      ibm-cloud.kubernetes.io/machine-type: b3c.4x16.encrypted
      ibm-cloud.kubernetes.io/os: UBUNTU_18_64
      ibm-cloud.kubernetes.io/region: us-south
      ibm-cloud.kubernetes.io/sgx-enabled: "false"
      ibm-cloud.kubernetes.io/worker-id: kube-br9v078d0qi43m0e31n0-remkohdevik-default-000002e0
      ibm-cloud.kubernetes.io/worker-pool-id: br9v078d0qi43m0e31n0-3108b12
      ibm-cloud.kubernetes.io/worker-pool-name: default
      ibm-cloud.kubernetes.io/worker-version: 1.16.10_1533
      ibm-cloud.kubernetes.io/zone: dal13
      ibm.role: worker_private
      kubernetes.io/arch: amd64
      kubernetes.io/hostname: 10.187.222.146
      kubernetes.io/os: linux
      privateVLAN: "2847992"
      publicVLAN: "2847990"
    name: kube-br9v078d0qi43m0e31n0-remkohdevik-default-000002e0-worker-private
    resourceVersion: "2295"
    uid: c2fcd831-a36b-11ea-97f2-06edddc7e672
  spec:
    expectedIPs:
    - 10.187.222.146
    interfaceName: eth0
    node: kube-br9v078d0qi43m0e31n0-remkohdevik-default-000002e0
- apiVersion: projectcalico.org/v3
  kind: HostEndpoint
  metadata:
    creationTimestamp: 2020-05-31T18:22:54Z
    labels:
      arch: amd64
      beta.kubernetes.io/arch: amd64
      beta.kubernetes.io/instance-type: b3c.4x16.encrypted
      beta.kubernetes.io/os: linux
      failure-domain.beta.kubernetes.io/region: us-south
      failure-domain.beta.kubernetes.io/zone: dal13
      ibm-cloud.kubernetes.io/encrypted-docker-data: "true"
      ibm-cloud.kubernetes.io/external-ip: 150.238.93.100
      ibm-cloud.kubernetes.io/ha-worker: "true"
      ibm-cloud.kubernetes.io/iaas-provider: softlayer
      ibm-cloud.kubernetes.io/internal-ip: 10.187.222.146
      ibm-cloud.kubernetes.io/machine-type: b3c.4x16.encrypted
      ibm-cloud.kubernetes.io/os: UBUNTU_18_64
      ibm-cloud.kubernetes.io/region: us-south
      ibm-cloud.kubernetes.io/sgx-enabled: "false"
      ibm-cloud.kubernetes.io/worker-id: kube-br9v078d0qi43m0e31n0-remkohdevik-default-000002e0
      ibm-cloud.kubernetes.io/worker-pool-id: br9v078d0qi43m0e31n0-3108b12
      ibm-cloud.kubernetes.io/worker-pool-name: default
      ibm-cloud.kubernetes.io/worker-version: 1.16.10_1533
      ibm-cloud.kubernetes.io/zone: dal13
      ibm.role: worker_public
      kubernetes.io/arch: amd64
      kubernetes.io/hostname: 10.187.222.146
      kubernetes.io/os: linux
      privateVLAN: "2847992"
      publicVLAN: "2847990"
    name: kube-br9v078d0qi43m0e31n0-remkohdevik-default-000002e0-worker-public
    resourceVersion: "2294"
    uid: e8c1fb2b-1a88-4cb4-a10e-67c9bdd28e1c
  spec:
    expectedIPs:
    - 150.238.93.100
    interfaceName: eth1
    node: kube-br9v078d0qi43m0e31n0-remkohdevik-default-000002e0
kind: HostEndpointList
metadata:
  resourceVersion: "40605"


$ calicoctl get NetworkPolicy --all-namespaces -o wide
NAMESPACE     NAME                               ORDER   SELECTOR                                                                       
kube-system   knp.default.kubernetes-dashboard   1000    projectcalico.org/orchestrator == 'k8s' && k8s-app == 'kubernetes-dashboard'   


$ calicoctl get GlobalNetworkPolicy -o wide
NAME    ORDER    SELECTOR
allow-all-outbound          1900     ibm.role in { 'worker_public', 'master_public' }   
allow-all-private-default   100000   ibm.role == 'worker_private'                       
allow-bigfix-port           1900     ibm.role in { 'worker_public', 'master_public' }   
allow-icmp                  1500     ibm.role in { 'worker_public', 'master_public' }   
allow-node-port-dnat        1500     ibm.role == 'worker_public'                        
allow-sys-mgmt              1950     ibm.role in { 'worker_public', 'master_public' }   
allow-vrrp                  1500     ibm.role == 'worker_public' 


$ echo 'apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
  namespace: advanced-policy-demo
spec:
  podSelector:
    matchLabels:
      app: helloworld
  ingress:
    - from:
      - podSelector:
          matchLabels: {}' > helloworld-policy-allow-to-helloworld.yaml