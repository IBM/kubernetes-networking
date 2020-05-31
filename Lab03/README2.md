# Lab 03 - Extra

Allocation of an external IP address requires the system to create an external IP address, a forwarding rule, a target proxy, a backend service, and possibly an instance group. Once the IP address has been allocated you can connect to your service through it, assign it a domain name and distribute it to clients.

## Ingress Subdomain

By default, IKS created an Ingress subdomain already when you created the cluster. You can also create a new Ingress subdomain or NLB hostname in addition,
```
% kubectl config current-context
% ibmcloud ks nlb-dns create classic -cluster remkohdev-iks116-3x-cluster --ip 169.48.75.82
OK
NLB hostname was created as remkohdev-iks116-3x-clu-2bef1f4b4097001da9502000c44fc2b2-0001.us-south.containers.appdomain.cloud
```

## Resources

- [Components and architecture of an NLB 1.0](https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-about#v1_planning).
- [Choosing an app exposure service, choosing a deployment pattern for classic clusters](https://cloud.ibm.com/docs/containers?topic=containers-cs_network_planning#pattern_public).
- [Quick start for load balancers](https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-qs)
- [Classic: Setting up DSR load balancing with an NLB 2.0 (beta)](https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-v2)