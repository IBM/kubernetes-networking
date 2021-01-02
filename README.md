# Kubernetes Networking

In this Kubernetes Networking workshop, you will deploy a `helloworld` application from `https://github.com/remkohdev/helloworld` and apply different service types and networking policies to controll access to the cluster and application.

# Labs

1. [Setup](docs/setup.md)
2. [Services](docs/services.md)
3. [ClusterIP](docs/clusterip.md)
4. [NodePort](docs/nodeport.md)
5. [Loadbalancer and NLB](docs/loadbalancer.md),
6. [ExternalName](docs/externalname.md)
7. Lab04 - Ingress and Application Load Balancer (ALB), see [Lab04](Lab04/README.md)
8. Lab05 - Kubernetes Network Policy and Calico, see [Lab05](Lab05/README.md)

# Prerequisites

1. Kubernetes Cluster
    - For Lab01 and Lab02 about Service types `ClusterIP` and `NodePort` you can use a single worker node Kubernetes cluster.
    - For Lab02 and Lab04 about Service type `LoadBalancer` and `Ingress` you need to have a Kubernetes cluster with a minimum of two worker nodes.
    - For Lab05 about `Kubernetes Network Policy` and `Calico`, you need.
1. Client, recommended to use IBM Cloud shell at https://shell.cloud.ibm.com.

# Todo

- [ ] Understanding the DNS Operator, https://docs.openshift.com/container-platform/4.5/networking/dns-operator.html, https://docs.openshift.com/container-platform/4.5/networking/cluster-network-operator.html
- [ ] 