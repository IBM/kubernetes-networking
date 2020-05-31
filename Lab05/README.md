# Lab05 - Kubernetes Network Policy and Calico 

Every IBM Cloud Kubernetes Service cluster is set up with a network plug-in called Calico. Default network policies are set up to secure the public network interface of every worker node in the cluster.

You can use Calico and Kubernetes to create network policies for a cluster. With Kubernetes network policies, you can specify the network traffic that you want to allow or block to and from a pod within a cluster. To set more advanced network policies such as blocking inbound (ingress) traffic to network load balancer (NLB) services, use Calico network policies.

## Kubernetes Network Policies

When a Kubernetes network policy is applied, it is automatically converted into a Calico network policy so that Calico can apply it as an Iptables rule.

Both incoming and outgoing network traffic can be allowed or blocked based on protocol, port, and source or destination IP addresses. Traffic can also be filtered based on pod and namespace labels.

## Calico Network Policies

Calico network policies are a superset of the Kubernetes network policies and are applied by using calicoctl commands. Calico policies add the following features:
- Allow or block network traffic on specific network interfaces regardless of the Kubernetes pod source or destination IP address or CIDR.
- Allow or block network traffic for pods across namespaces.
- Block inbound traffic to Kubernetes LoadBalancer or NodePort services.

Calico enforces these policies, including any Kubernetes network policies that are automatically converted to Calico policies, by setting up Linux Iptables rules on the Kubernetes worker nodes. Iptables rules serve as a firewall for the worker node to define the characteristics that the network traffic must meet to be forwarded to the targeted resource.

# Adopt a zero trust network model

Adopting a zero trust network model is best practice for securing workloads and hosts in your cloud-native strategy.

Use Case:
- create helloworld pod
- create helloworld proxy
- access helloworld direct and via proxy
- leave nodeport open
- apply network policy to block all traffic
- apply network policy to allow traffic via proxy

