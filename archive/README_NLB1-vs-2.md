# Lab03 - Extra

See also [Controlling inbound traffic to NLB or NodePort services](https://cloud.ibm.com/docs/containers?topic=containers-network_policies#block_ingress).


See [Comparison of basic and DSR load balancing in version 1.0 and 2.0 NLBs](https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-about#comparison)

When you create an NLB, you can choose a version 1.0 NLB, which performs basic load balancing, or version 2.0 NLB, which performs direct server return (DSR) load balancing. Version 1.0 and 2.0 NLBs are both Layer 4 load balancers that exist in the Linux kernel space. 

Both versions run inside the cluster, and use worker node resources. Therefore, the available capacity of the NLBs is always dedicated to your own cluster. Additionally, both versions of NLBs do not terminate the connection. Instead, they forward connections to an app pod.

When a client sends a request to your app, the NLB routes request packets to the worker node IP address where an app pod exists. Version 1.0 NLBs use network address translation (NAT) to rewrite the request packet's source IP address to the IP of worker node where a load balancer pod exists. When the worker node returns the app response packet, it uses that worker node IP where the NLB exists. The NLB must then send the response packet to the client.