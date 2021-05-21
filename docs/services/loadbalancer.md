# Loadbalancer and Network Load Balancer (NLB) 1.0

## Pre-requisites

Finish the [Services](services.md), [ClusterIP](clusterip.md), and [NodePort](nodeport.md) labs.

## LoadBalancer

In the previous labs, you created a service for the `helloworld` application with a default clusterIP and then added a NodePort to the Service, which proxies requests to the Service resource. But in a production environment, you still need a type of load balancer, whether client requests are internal or external coming in over the public network. 

The `LoadBalancer` service in Kubernetes configures an Open Systems Interconnection (OSI) model Layer 4 (L4) load balancer, which forwards and balances traffic from the internet to your backend application.

To use a load balancer for distributing client traffic to the nodes in a cluster, you need a public IP address that the clients can connect to, and you need IP addresses on the nodes themselves to which the load balancer can forward the requests. 

When you create a standard cluster, IKS and ROKS automatically provision a portable public subnet and a portable private subnet. The portable public subnet provides 5 usable IP addresses. 1 portable public IP address is used by the default public `Ingress ALB`. The remaining 4 portable public IP addresses can be used to expose single apps using layer 4 (L4) TCP/UDP Network Load Balancer (NLB).

The portable public and private IP addresses are static floating IPs pointing to worker nodes. A `Keepalived` daemon  constantly monitors the IP, and automatically moves the IP to another worker node if the worker node is removed.  

See: [Classic: About network load balancers (NLBs)](https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-about#v1_planning).

Services of type `LoadBalancer` have some limitations. They cannot do TLS termination, do virtual hosts or path-based routing, so you can’t use a single load balancer to proxy to multiple services. These limitations led to the addition in Kubernetes v1.2 of a separate kubernetes resource called `Ingress` and `Route` (on OpenShift).

Because, the creation of the load balancer happens asynchronously with the creation of the Service, you might have to wait until both the Service and the load balancer have been created.

## Load Balancer on IBM Cloud

The LoadBalancer service type is implemented differently depending on your cluster's infrastructure provider. On IBM Kubernetes Service (IKS) and Red Hat OpenShift Kubernetes Service (ROKS) on IBM Cloud, `classic clusters` implement by default a Network Load Balancer (NLB) 1.0. 

Version 1.0 NLBs use `Network Address Translation (NAT)` to rewrite the request packet's source IP address to the IP of worker node where a load balancer pod exists. 

![NLB 1.0](images/ks_loadbalancer_nlb1.png)

Version 2.0 NLBs doesn't use NAT but `IP over IP (IPIP)` to encapsulate the original request packet into another packet, which preserves the client IP as its source IP address. The worker node then uses `Direct Server Return (DSR)` to send the app response packet back to the client IP.

![NLB 2.0](images/ks_loadbalancer_nlb2.png)

## Load Balancing Methods on IBM Cloud

Before we create the `LoadBalancer` service for the `helloworld` application, review some of the Load Balancing options on Kubernetes in IBM Cloud:

- **`NodePort`** exposes the app via a port and public IP address on a worker node using `kube-proxy`.
- **`LoadBalancer NLB v1.0`** uses basic load balancing that exposes the app with an IP address or a subdomain.
- **`LoadBalancer NLB v2.0`**, uses Direct Server Return (DSR) load balancing to expose the app with an IP address or a subdomain, and supports SSL/TLS termination.
- **`Istio Ingress Gateway + NLB`** uses basic load balancing that exposes the app with a subdomain and uses Istio routing rules.
- **`Ingress with public ALB`** uses HTTPS load balancing that exposes the app with a subdomain and uses custom routing rules and SSL/TLS termination for multiple apps. You can customize the ALB routing rules with [annotations](https://cloud.ibm.com/docs/containers?topic=containers-ingress_annotation).
- **`Custom Ingress + NLB`** uses HTTPS load balancing with a custom Ingress that exposes the app with the IBM-provided ALB subdomain and uses custom routing rules.

## Create a LoadBalancer

In the previous lab, you already created a `NodePort` Service. 

```
$ oc get svc -n $MY_NS

NAME         TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
helloworld   NodePort   172.21.86.16   <none>        8080:32387/TCP   12m
```

Patch the service for `helloworld` and change the type to `LoadBalancer`.

```
oc patch svc helloworld -p '{"spec": {"type": "LoadBalancer"}}' -n $MY_NS
```

The `TYPE` should be set to `LoadBalancer` now, and an `EXTERNAL-IP` should be assigned.

```
$ oc get svc helloworld -n $MY_NS

NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
helloworld   LoadBalancer   172.21.86.16   169.47.155.242   8080:32387/TCP   12m
```

To access the Service of the `helloworld` from the public internet, you can use the public IP address of the NLB and the assigned NodePort of the service in the format `<IP_address>:<NodePort>`.

```
PUBLIC_IP=$(oc get svc helloworld -n $MY_NS --output json | jq -r '.status.loadBalancer.ingress[0].ip')
echo $PUBLIC_IP

NODE_PORT=$(oc get svc helloworld -n $MY_NS --output json | jq -r '.spec.ports[0].nodePort' )
echo $NODE_PORT
```

Access the `helloworld` app in a browser or with Curl,

```
$ curl -L -X POST "http://$PUBLIC_IP:$NODE_PORT/api/messages" -H 'Content-Type: application/json' -d '{ "sender": "world2" }'

{"id":"0ebdc166-32cd-4d0d-93b6-f278e4405c6f","sender":"world2","message":"Hello world2 (direct)","host":null}
```

## Next

Next, go to [ExternalName](externalname.md).
