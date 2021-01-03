# Kubernetes Networking

In this Kubernetes Networking workshop, you will use different ways to control traffic on a Kubernetes cluster, from Service types, load balancing and Ingress, Network Policy and Calico, and VPC Gen2.

## Labs

1. [Setup](docs/setup.md)
2. [Services](docs/services.md)
3. [ClusterIP](docs/clusterip.md)
4. [NodePort](docs/nodeport.md)
5. [Loadbalancer NLB](docs/loadbalancer.md),
6. [ExternalName](docs/externalname.md)
7. [Ingress ALB](docs/ingress-alb.md)
8. [Network Policy](docs/networkpolicy.md) 
9. [Calico](docs/calico.md)
10. [VPC Gen2](docs/vpcgen2.md)
11. [Airgap](docs/airgap.md)

## Prerequisites

1. Kubernetes Cluster
2. ClientTerminal

## Technologies

This workshop was tested using the following technologies:

* IBM Cloud Kubernetes Service (IKS) version 1.19, 2 worker nodes, flavor u3c.2x4
* Calico client version v3.17.1
* Calico cluster version v3.16.5
* ibmcloud version 1.3.0
* ibmcloud container-service/kubernetes-service   1.0.28
* vpc-infrastructure/infrastructure-service 0.7.5
* kubectl version 1.19

Contributors:
* Remko de Knikker, [remkohdev](https://github.com/remkohdev),
* Tim Robinson, [timroster](https://github.com/timroster),