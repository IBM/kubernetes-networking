# Lab06 - Calico

https://cloud.ibm.com/docs/containers?topic=containers-network_policies

The public network interface for worker nodes is protected by predefined Calico network policy settings that are configured on every worker node during cluster creation. By default, all outbound network traffic is allowed for all worker nodes. Inbound network traffic is blocked except for a few ports. 

Every IBM Cloud™ Kubernetes Service cluster is set up with a network plug-in called Calico. Default network policies are set up to secure the public network interface of every worker node in the cluster.

You can use Calico and Kubernetes to create network policies for a cluster. With Kubernetes network policies, you can specify the network traffic that you want to allow or block to and from a pod within a cluster. To set more advanced network policies such as blocking inbound (ingress) traffic to network load balancer (NLB) services, use Calico network policies.