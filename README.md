# Kubernetes Networking

In this Kubernetes Networking workshop, you will use the Guestbook application from https://github.com/IBM/guestbook.git.

# Prerequisites

For the Service types `ClusterIP` and `NodePort` you can use a single worker node cluster, but the types `LoadBalancer` and `Ingress` require 

Clone the guestbook application,
```
$ git clone https://github.com/IBM/guestbook.git
$ cd guestbook
```

Login to your cluster,
```
$ ibmcloud login -sso
$ CLUSTER_NAME=<clustername>
$ ibmcloud ks cluster config --cluster $CLUSTER_NAME
$ kubectl config current-context
```


# Labs

1. Lab01 - Services and ClusterIP, see [Lab01](Lab01/README.md)
2. Lab02 - NodePort, see [Lab02](Lab02/README.md)
3. Lab03 - Loadbalancer and Network Load Balancer (NLB) 1.0, see [Lab03](Lab03/README.md),
4. Lab04 - Ingress and Application Load Balancer (ALB), see [Lab04](Lab04/README.md)

