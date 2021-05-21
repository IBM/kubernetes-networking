# Ingress Extra

Allocation of an external IP address requires the system to create an external IP address, a forwarding rule, a target proxy, a backend service, and possibly an instance group. Once the IP address has been allocated you can connect to your service through it, assign it a domain name and distribute it to clients.

## Ingress Subdomain

By default, IKS created an Ingress subdomain already when you created the cluster. You can also create a new Ingress subdomain or NLB hostname in addition,
```
% kubectl config current-context
% ibmcloud ks nlb-dns create classic -cluster remkohdev-iks116-3x-cluster --ip 169.48.75.82
OK
NLB hostname was created as remkohdev-iks116-3x-clu-2bef1f4b4097001da9502000c44fc2b2-0001.us-south.containers.appdomain.cloud
```

There is also the option to set up Direct Server Return (DSR) load balancing with an NLB 2.0 see [here](https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-v2).

When you create a Kubernetes LoadBalancer service for an app in your IKS cluster, a layer 7 Virtual Private Cloud (VPC) load balancer is automatically created in your VPC outside of your cluster. The VPC load balancer is multizonal and routes requests for your app through the private NodePorts that are automatically opened on your worker nodes. 

In a `free cluster` on IKS, the cluster's worker nodes are connected to an IBM-owned public VLAN and private VLAN by default. Because IBM controls the VLANs, subnets, and IP addresses, you cannot create multizone clusters or add subnets to your cluster, and can use only NodePort services to expose your app.

If your cluster is a free cluster or only has 1 worker node, the EXTERNAL-IP will say `pending` because in vain the cluster is waiting on the asynchronous response.

- The portable public subnet provides 5 usable IP addresses. 1 portable public IP address is used by the default public Ingress ALB. The remaining 4 portable public IP addresses can be used to expose single apps to the internet by creating public network load balancer services, or NLBs.
- The portable private subnet provides 5 usable IP addresses. 1 portable private IP address is used by the default private Ingress ALB. The remaining 4 portable private IP addresses can be used to expose single apps to a private network by creating private load balancer services, or NLBs.

## Resources

- [Components and architecture of an NLB 1.0](https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-about#v1_planning).
- [Choosing an app exposure service, choosing a deployment pattern for classic clusters](https://cloud.ibm.com/docs/containers?topic=containers-cs_network_planning#pattern_public).
- [Quick start for load balancers](https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-qs)
- [Classic: Setting up DSR load balancing with an NLB 2.0 (beta)](https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-v2)
