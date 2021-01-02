# Kubernetes Networking

In this Kubernetes Networking workshop, you will deploy a `helloworld` application from `https://github.com/remkohdev/helloworld` and apply different service types and networking policies to controll access to the cluster and application.

## Prerequirements

1. Free IBM Cloud account, to create a new IBM Cloud account go [here](https://ibm.github.io/workshop-setup/NEWACCOUNT/).
1. Free `Pay-As-You-Go` account. To upgrade a free IBM Cloud account, go [here](https://ibm.github.io/workshop-setup/PAYASYOUGO/).
1. CognitiveLabs.ai account, to access a client terminal at CognitiveLabs.ai, go [here](https://ibm.github.io/workshop-setup/COGNITIVECLASS/).
1. Kubernetes cluster v1.18:
    - for labs in `Services`, `ClusterIP`, and `NodePort`, you need a Kubernetes cluster with at least 1 worker node.
    - for labs in  `LoadBalancer` and `Ingress` you need a Kubernetes cluster with at least 2 worker nodes.
    - for labs in `Network Policy` and `Calico`, you need a Kubernetes cluster with at least 2 worker nodes.
    - for labs in `VPC Gen2`, you need a Kubernetes cluster with at least 1 worker node.

Go to [Setup](setup.md) for more details.

# Labs

1. [Setup](setup.md)
2. [Services](services.md)
3. [ClusterIP](clusterip.md)
4. [NodePort](nodeport.md)
5. [Loadbalancer NLB](loadbalancer.md)
6. [ExternalName](externalname.md)
7. [Ingress ALB](ingress-alb.md)
8. [Network Policy](networkpolicy.md) 
9. [Calico](calico.md)
10. [VPC Gen2](vpcgen2.md)
