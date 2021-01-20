


find your cloud shell name
```
$ ls -al /tmp/ic
..
cloudshell-a066b1e2-1975-41d3-a316-d9e92d917e90-1-7f5b46f8v96r5-1

$ SHELL=cloudshell-a066b1e2-1975-41d3-a316-d9e92d917e90-1-7f5b46f8v96r5-1
```

Find the cluster's admin folder
```
$ ls -al /tmp/ic/$SHELL/.bluemix/plugins/container-service/clusters/
..
remkohdev-iks116-2n-cluster-br9v078d0qi43m0e31n0
remkohdev-iks116-2n-cluster-br9v078d0qi43m0e31n0-admin

$ ADMINDIR=remkohdev-iks116-2n-cluster-br9v078d0qi43m0e31n0-admin

/tmp/ic/$SHELL/.bluemix/plugins/container-service/clusters/$ADMINDIR/calicoctl.cfg

$ mv /tmp/ic/$SHELL/.bluemix/plugins/container-service/clusters/$ADMINDIR/calicoctl.cfg /etc/calico

$ calicoctl get nodes
NAME                                                     
kube-br9v078d0qi43m0e31n0-remkohdevik-default-0000014b   
kube-br9v078d0qi43m0e31n0-remkohdevik-default-000002e0

$ calicoctl version
Client Version:    v3.10.0
Git commit:        7968b525
Cluster Version:   v3.9.5
Cluster Type:      k8s,bgp
```

View network policy

```
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
```

```
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
```

