# Ingress and Application Load Balancer (ALB)

## Pre-requisites

Finish the [Services](services.md), [ClusterIP](clusterip.md), [NodePort](nodeport.md) and [LoadBalancer](loadbalancer.md) labs. This should provide you with:

* Logged in to IBM Cloud account,
* Connected to Kubernetes cluster,
* Guestbook Deployment,
* Guestbook Service of type LoadBalancer.

## Network Administration

When you create a standard cluster in IBM Cloud Kubernetes Service (IKS), a portable public subnet and a portable private subnet for the VLAN are automatically provisioned.

To retrieve the cluster id, you need the cluster name. If you do not know the cluster name already, you can list all clusters in the active account,

```
ibmcloud ks clusters
```

Create an environment variable and set the cluster name,

```
KS_CLUSTER_NAME=<cluster name>
echo $KS_CLUSTER_NAME
```

The portable public subnet for your cluster on IBM Cloud provides 5 usable IP addresses. 1 portable public IP address is used by the default public Ingress ALB. The remaining 4 portable public IP addresses can be used to expose single apps to the internet by creating public Network Load Balancer (NLB) services.

To list all of the portable IP addresses in the Kubernetes cluster, both used and available, you can retrieve the following `ConfigMap` in the `kube-system` namespace listing the resources of the subnets,

```
$ oc get cm ibm-cloud-provider-vlan-ip-config -n kube-system -o yaml

apiVersion: v1
data:
  cluster_id: c09nq3aw0cv06eckrvsg
  reserved_private_ip: ""
  reserved_private_vlan_id: ""
  reserved_public_ip: ""
  reserved_public_vlan_id: ""
  vlanipmap.json: |-
    {
      "vlans": [
        {
          "id": "2936320",
          "subnets": [
            {
              "id": "2448002",
              "ips": [
                "10.216.29.66",
                "10.216.29.67",
                "10.216.29.68",
                "10.216.29.69",
                "10.216.29.70"
              ],
              "is_public": false,
              "is_byoip": false,
              "cidr": "10.216.29.64/29"
            }
          ],
          "zone": "wdc04",
          "region": "us-east"
        },
        {
          "id": "2936318",
          "subnets": [
            {
              "id": "2390180",
              "ips": [
                "169.47.155.242",
                "169.47.155.243",
                "169.47.155.244",
                "169.47.155.245",
                "169.47.155.246"
              ],
              "is_public": true,
              "is_byoip": false,
              "cidr": "169.47.155.240/29"
            }
          ],
          "zone": "wdc04",
          "region": "us-east"
        }
      ],
      "vlan_errors": [],
      "reserved_ips": []
    }
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"cluster_id":"c09nq3aw0cv06eckrvsg","reserved_private_ip":"","reserved_private_vlan_id":"","reserved_public_ip":"","reserved_public_vlan_id":"","vlanipmap.json":"{\n  \"vlans\": [\n    {\n      \"id\": \"2936320\",\n      \"subnets\": [\n        {\n          \"id\": \"2448002\",\n          \"ips\": [\n            \"10.216.29.66\",\n            \"10.216.29.67\",\n            \"10.216.29.68\",\n            \"10.216.29.69\",\n            \"10.216.29.70\"\n          ],\n          \"is_public\": false,\n          \"is_byoip\": false,\n          \"cidr\": \"10.216.29.64/29\"\n        }\n      ],\n      \"zone\": \"wdc04\",\n      \"region\": \"us-east\"\n    },\n    {\n      \"id\": \"2936318\",\n      \"subnets\": [\n        {\n          \"id\": \"2390180\",\n          \"ips\": [\n            \"169.47.155.242\",\n            \"169.47.155.243\",\n            \"169.47.155.244\",\n            \"169.47.155.245\",\n            \"169.47.155.246\"\n          ],\n          \"is_public\": true,\n          \"is_byoip\": false,\n          \"cidr\": \"169.47.155.240/29\"\n        }\n      ],\n      \"zone\": \"wdc04\",\n      \"region\": \"us-east\"\n    }\n  ],\n  \"vlan_errors\": [],\n  \"reserved_ips\": []\n}"},"kind":"ConfigMap","metadata":{"annotations":{},"labels":{"addonmanager.kubernetes.io/mode":"Reconcile","kubernetes.io/cluster-service":"true"},"name":"ibm-cloud-provider-vlan-ip-config","namespace":"kube-system"}}
  creationTimestamp: "2021-01-29T03:29:56Z"
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: ibm-cloud-provider-vlan-ip-config
  namespace: kube-system
  resourceVersion: "14367"
  selfLink: /api/v1/namespaces/kube-system/configmaps/ibm-cloud-provider-vlan-ip-config
  uid: b2a8098e-61c5-4ced-a589-b2df1dcb6275
```

One of the public IP addresses on the public VLAN's subnet is assigned to the Network Load Balancer (NLB).

List the registered NLB host name and IP address for your cluster (depending on the IBM Cloud CLI version, use `--json` or `--output json`),

```
$ ibmcloud ks nlb-dns ls --cluster $KS_CLUSTER_NAME --json

[
    {
        "clusterID": "c09nq3aw0cv06eckrvsg",
        "nlbIP": "[\"169.47.155.244\"]",
        "nlbIPArray": [
            "169.47.155.244"
        ],
        "nlbType": "public",
        "nlbHost": "dte-ocp44-on2g3r-915b3b336cabec458a7c7ec2aa7c625f-0000.us-east.containers.appdomain.cloud",
        "secretNamespace": "openshift-ingress",
        "nlbMonitorState": "None",
        "nlbSslSecretName": "dte-ocp44-on2g3r-915b3b336cabec458a7c7ec2aa7c625f-0000",
        "nlbSslSecretStatus": "created"
    }
]
```

And retrieve the NodePort via,

```
NODE_PORT=$(oc get svc helloworld -n $MY_NS --output json | jq -r '.spec.ports[0].nodePort' )
echo $NODE_PORT
```

You see that the portable IP address was assigned to the NLB of the LoadBalancer service.

## Ingress ALB

`Ingress` is a reverse-proxy load balancer and Kubernetes API object that manages external access to services in a cluster. You can also use `Ingress` to expose multiple app services to a public or private network by using a single unique route. The Ingress API also supports TLS termination, virtual hosts, and path-based routing.

Ingress consists of **three components**:

* Ingress resources,
* Application load balancers (ALBs),
* A load balancer to handle incoming requests across zones.

To expose an app with Ingress, create a Kubernetes service of type `LoadBalancer` and register this Service with Ingress by defining an Ingress resource. One Ingress resource is required per namespace where you have apps that you want to expose. 

The Ingress resource is a Kubernetes resource that defines the rules for how to route incoming requests for apps. The Ingress resource also specifies the path to your app services. When you created a standard IKS cluster, an Ingress subdomain is already registered by default for your cluster. The paths to your app services are appended to the public route.

In a standard cluster on IKS, the Ingress Application Load Balancer (ALB) is a layer 7 (L7) load balancer which implements the [`NGINX` Ingress controller](https://docs.nginx.com/nginx-controller/).

In a [Red Hat OpenShift Kubernetes Service (ROKS) cluster](https://cloud.ibm.com/docs/openshift?topic=openshift-ingress-about-roks4), the router is a layer 7 load balancer which implements an [`HAProxy` Ingress controller](https://github.com/haproxytech/kubernetes-ingress).

![Multi-zone cluster Ingress ALB traffic](images/ks_ingress_multizone.png)

## Create an Ingress Resource for the HelloWorld App

Instead of using `<external-ip>:<nodeport>` to access the HelloWorld app, we want to access our HelloWorld aplication via the URL `<Ingress-subdomain>:<nodeport>/<path>`. 

To configure your Ingress resource, you need the Ingress Subdomain and Ingress Secret of your cluster. Both were already created by IKS when you created the cluster. 

```
INGRESS_SUBDOMAIN=$(ibmcloud ks nlb-dns ls --cluster $KS_CLUSTER_NAME --json | jq -r '.[0].nlbHost')
echo $INGRESS_SUBDOMAIN

INGRESS_SECRET=$(ibmcloud ks nlb-dns ls --cluster $KS_CLUSTER_NAME --json | jq -r '.[0].nlbSslSecretName')
echo $INGRESS_SECRET
```

Or,

```
INGRESS_SUBDOMAIN=$(ibmcloud ks cluster get --show-resources -c $KS_CLUSTER_NAME --json | jq -r '.ingressHostname')
echo $INGRESS_SUBDOMAIN

INGRESS_SECRET=$(ibmcloud ks cluster get --show-resources -c $KS_CLUSTER_NAME --json | jq -r '.ingressSecretName')
echo $INGRESS_SECRET
```

Create the Ingress specification and change the `hosts` and `host` to the `Ingress Subdomain` of your cluster, and change the `secretName` to the value `Ingress Secret` of your cluster. Potentially, you can use annotations like `rewrite path` to customize an Ingress resource. See [https://cloud.ibm.com/docs/containers?topic=containers-ingress_annotation](https://cloud.ibm.com/docs/containers?topic=containers-ingress_annotation).

Find the Kubernetes version,

```
$ oc get nodes -o wide

NAME             STATUS   ROLES           AGE   VERSION           INTERNAL-IP      EXTERNAL-IP      OS-IMAGE   KERNEL-VERSION                CONTAINER-RUNTIME
10.183.200.134   Ready    master,worker   19h   v1.17.1+40d7dbd   10.183.200.134   169.47.169.205   Red Hat    3.10.0-1160.11.1.el7.x86_64   cri-o://1.17.5-11.rhaos4.4.git7f979af.el7
10.183.200.181   Ready    master,worker   19h   v1.17.1+40d7dbd   10.183.200.181   169.47.169.203   Red Hat    3.10.0-1160.11.1.el7.x86_64   cri-o://1.17.5-11.rhaos4.4.git7f979af.el7
```

In version 1.17 and 1.18 syntax,

```
echo "apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: helloworld-ingress
  annotations:
    kubernetes.io/ingress.class: \"public-iks-k8s-nginx\"
spec:
  tls:
  - hosts:
    - $INGRESS_SUBDOMAIN
    secretName: $INGRESS_SECRET
  rules:
  - host: $INGRESS_SUBDOMAIN
    http:
      paths:
      - path: /
        backend:
          serviceName: helloworld
          servicePort: 8080" > helloworld-ingress.yaml
```

In version 1.19 syntax,

```
echo "apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: helloworld-ingress
  annotations:
    kubernetes.io/ingress.class: "public-iks-k8s-nginx"
spec:
  tls:
  - hosts:
    - $INGRESS_SUBDOMAIN
    secretName: $INGRESS_SECRET
  rules:
  - host: $INGRESS_SUBDOMAIN
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: helloworld
            port: 
              number: 8080" > helloworld-ingress.yaml
```

The above resource will create an access path to the helloworld at `http://$INGRESS_SUBDOMAIN:$PORT/`. 

You can further [customize Ingres routing with annotations](https://cloud.ibm.com/docs/containers?topic=containers-ingress_annotation) to customize the ALB settings, TLS settings, request and response annocations, service limits, user authentication, or error actions. 

Make sure, the values for the `hosts`, `secretName` and `host` are set correctly to match the values of the Ingress Subdomain and Secret of your cluster. Edit the `helloworld-ingress.yaml` file to make the necessary changes,

Then create the Ingress for helloworld,

```
$ oc create -f helloworld-ingress.yaml -n $MY_NS

ingress.networking.k8s.io/helloworld-ingress created
```

To find the service port again,

```
NODE_PORT=$(oc get svc helloworld -n $MY_NS --output json | jq -r '.spec.ports[0].nodePort' )
echo $NODE_PORT
```

Try to access the `helloworld` API and the proxy using the Ingress Subdomain with the path to the service,

```
$ curl -L -X POST "http://$INGRESS_SUBDOMAIN:$NODE_PORT/api/messages" -H 'Content-Type: application/json' -d '{ "sender": "world3" }'

{"id":"c806432d-0f84-45bb-a654-0b6be0146044","sender":"world3","message":"Hello world3 (direct)","host":null}
```

If you instead want to use subdomain paths instead of URI paths,

```
echo "apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: helloworld-ingress
  annotations:
    kubernetes.io/ingress.class: \"public-iks-k8s-nginx\"
spec:
  tls:
  - hosts:
    - $INGRESS_SUBDOMAIN
    secretName: $INGRESS_SECRET
  rules:
    - host: >-
        $INGRESS_SUBDOMAIN
      http:
        paths:
          - backend:
              serviceName: helloworld
              servicePort: 8080
    - host: >-
        hello.$INGRESS_SUBDOMAIN
      http:
        paths:
          - backend:
              serviceName: helloworld
              servicePort: 8080" > helloworld-ingress-subdomain.yaml
```

Delete the previous Ingress resource and create the Ingress resource using subdomain paths.

```
oc get ingress -n $MY_NS
oc delete ingress helloworld-ingress -n $MY_NS
oc create -f helloworld-ingress-subdomain.yaml -n $MY_NS
```

Try to access the application using the subdomain,

```
$ curl -L -X POST "http://hello.$INGRESS_SUBDOMAIN/api/messages" -H 'Content-Type: application/json' -d '{ "sender": "world4" }'

{"id":"ea9c00e9-190a-4d83-ab8a-cf6e81c1bb10","sender":"world4","message":"Hello world4 (direct)","host":null}
```

## Next

Next, go to [Route](route.md).
