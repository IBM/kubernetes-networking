# Kubernetes Networking

In this Kubernetes Networking workshop, you will use different ways to control traffic on a Kubernetes cluster, from Service types, load balancing and Ingress, Network Policy and Calico, and VPC Gen2. 

This series of workshops on Kubernetes Networking is accompanied by a [lecture on Kubernetes Networking](https://raw.githubusercontent.com/remkohdev/kubernetes-networking/master/pdf/KubernetesNetworking-Lecture.pdf).

Kubernetes Networking is part of the series `Kubernetes Security`, which includes:

1. Kubernetes Security,
2. Kubernetes Networking,
3. Kubernetes Storage,
4. Kubernetes Automation (DevOps, IaC, CI/CD).

## Prerequirementss

* Free IBM Cloud account, to create a new IBM Cloud account go [here](https://ibm.github.io/workshop-setup/NEWACCOUNT/).
* Free `Pay-As-You-Go` account. To upgrade a free IBM Cloud account, go [here](https://ibm.github.io/workshop-setup/PAYASYOUGO/).
* CognitiveLabs.ai account, to access a client terminal at CognitiveLabs.ai, go [here](https://ibm.github.io/workshop-setup/COGNITIVECLASS/).
* Kubernetes standard cluster v1.18 with at least 2 worker nodes on a VLAN, a subnet with public IPs, external LoadBalancer (for details about VLAN, subnets and IPs, see [here](https://cloud.ibm.com/docs/containers?topic=containers-subnets) ). 

Go to [Setup](setup.md) for more details.

# Labs

1. [Setup](setup.md)
2. [Services](services.md)
3. [ClusterIP](clusterip.md)
4. [NodePort](nodeport.md)
5. [Loadbalancer NLB](loadbalancer.md)
6. [ExternalName](externalname.md)
7. [Ingress ALB](ingress-alb.md)
8. [Network Policy and Calico](networkpolicy.md) 
9.  [VPC Gen2](vpcgen2.md)
10. [Airgap](airgap.md)
