# Kubernetes Networking Extra

## Kubernetes Networking in-depth

The following image demonstrates how Kubernetes forwards public network traffic through kube-proxy and NodePort, LoadBalancer, or Ingress services in IBM Cloud Kubernetes Service.

![IKS public network traffic](images/cs_network_planning_ov-01.png)

The following image shows what features each of the different ServiceTypes supports on IBM Cloud.

![IKS public network traffic](images/kubernetes-network-service-types.png)

The services network is implemented by a kubernetes component called kube-proxy collaborating with a linux kernel module called netfilter to trap and reroute traffic sent to the cluster IP so that it is sent to a healthy pod instead

Connections and requests operate at OSI layer 4 (tcp) or layer 7 (http, rpc, etc). Netfilter rules are routing rules, and they operate on IP packets at layer 3. All routers, including netfilter, make routing decisions based more or less solely on information contained in the packet; generally where it is from and where it is going. So to describe this behavior in layer 3 terms: each packet destined for the service at10.3.241.152:80 that arrives at a node’s eth0 interface is processed by netfilter, matches the rules established for our service, and is forwarded to the IP of a healthy pod.

![Layer 3 Forwarding](images/kube-layer3-networking.png)

External clients that call into our pods has to make use of this same routing infrastructure. The cluster IP and port is the “front end”. The cluster IP of a service is only reachable from a node’s ethernet interface. How can we forward traffic from a publicly visible IP endpoint to an IP that is only reachable once the packet is already on a node?

One way is to examine the netfilter rules using the iptables utility. Any packet from anywhere that arrives on the node’s ethernet interface with a destination of 10.3.241.152:80 is going to match and get routed to a pod. So you could give clients the cluster IP or a friendly domain name, and then add a route to get the packets to one of the nodes.

But routers operating on layer 3 packets don’t know healthy services from unhealthy. Kube-proxy’s role is to actively manage netfilter. We can’t easily create a stable static route between the gateway router and the nodes using the service network (cluster IP).

## Kube-Proxy

Kubernetes relies on proxying to forward inbound traffic to backends. When you use the name of the service, it looks up the name in the cluster DNS provider and routes the request to the in-cluster IP address of the service.

A Kubernetes cluster runs a local Kubernetes network proxy, `kube-proxy`, as a daemon on each worker node in the kube-system namespace, which provides basic load balancing for services.

The default load balancing mode in Kubernetes is `iptables` and rules, a Linux kernel feature, to direct requests to the pods behind a service equally. The native method for load distribution in `iptables` mode is `random selection`. 

An older kube-proxy mode is `userspace`, which uses `round-robin` load distribution, allocating the next available pod on an IP list, then rotating the list.

Proxy modes:
- userspace
- iptables (default) (NLB 1.0 uses iptables)
- IPVS (NLB 2.0 uses IP Virtual Server (IPVS))

## DNS

A cluster-aware DNS server, such as [CoreDNS](https://coredns.io/), watches the Kubernetes API for new Services and creates a set of DNS records for each service. If DNS has been enabled throughout your cluster then all Pods should automatically be able to resolve Services by their DNS name.

Kubernetes also supports DNS SRV (DNS Service) records for named ports. If the "my-service.my-ns" Service has a port named "http" with the protocol set to TCP, you can do a DNS SRV query for _http._tcp.my-service.my-ns to discover the port number for "http", as well as the IP address.

The Kubernetes DNS server is the only way to access ExternalName Services.

Kubernetes offers a DNS cluster addon, which most of the supported environments enable by default. In Kubernetes version 1.11 and later, CoreDNS is recommended and is installed by default with kubeadm.

To verify if the CoreDNS deployment is available,

```
kubectl get deployment -n kube-system | grep dns

coredns               3/3    3    3    7h51m
coredns-autoscaler    1/1    1    1    7h51m

kubectl get configmap -n kube-system | grep dns

coredns               1    11d
coredns-autoscaler    1    11d
node-local-dns        1    11d
```

The `node-local-dns` is the `NodeLocal DNSCaching` agent for improved cluster DNS performance. [`NodeLocal DNSCache`](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/) is a feature introduced as stable in v1.18.

Next, go back to continue.

## Resources
- [Choosing an app exposure service](https://cloud.ibm.com/docs/containers?topic=containers-cs_network_planning)
- [ExternalName Service](https://kubernetes.io/docs/concepts/services-networking/service/#externalname)
- [supported protocols](https://kubernetes.io/docs/concepts/services-networking/service/#protocol-support)