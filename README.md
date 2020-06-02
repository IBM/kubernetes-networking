# Kubernetes Networking

In this Kubernetes Networking workshop, you will deploy a `helloworld` application from `https://github.com/remkohdev/helloworld` and apply different service types and networking policies to controll access to the cluster and application.

# Labs

1. Lab00 - Setup 
1. Lab01 - Services and ClusterIP, see [Lab01](Lab01/README.md)
2. Lab02 - NodePort, see [Lab02](Lab02/README.md)
3. Lab03 - Loadbalancer and Network Load Balancer (NLB) 1.0, see [Lab03](Lab03/README.md),
4. Lab04 - Ingress and Application Load Balancer (ALB), see [Lab04](Lab04/README.md)
5. Lab05 - Kubernetes Network Policy and Calico, see [Lab05](Lab05/README.md)


# Prerequisites

1. Kubernetes Cluster
    - For Lab01 and Lab02 about Service types `ClusterIP` and `NodePort` you can use a single worker node Kubernetes cluster.
    - For Lab02 and Lab04 about Service type `LoadBalancer` and `Ingress` you need to have a Kubernetes cluster with a minimum of two worker nodes.
    - For Lab05 about `Kubernetes Network Policy` and `Calico`, you need.
1. Client, recommended to use IBM Cloud shell at https://shell.cloud.ibm.com.
