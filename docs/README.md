# Kubernetes Networking

The Kubernetes Networking series consists of the following topics:

1. **Kubernetes Networking 101** (60 mins), you will use different ways to control traffic on a Kubernetes cluster with Service types. Start [here](services/services.md).
1. **Add an Ingress on OpenShift** (15 minutes), add an Ingress and Route to expose a Service, you will use different types of TLS termination to secure Routes on OpenShift: edge, passthrough and reencrypt. Start [here](route/route.md).
1. **Network Policies and Calico** (15 minutes), create a Network Policy and use Calico. Start [here](calico/networkpolicy.md).
1. **Kubernetes Network Security using a Virtual Private Cloud (VPC)** (90 mins), you will deploy a guestbook application to a Kubernetes cluster in a Virtual Private Cloud (VPC) Gen2, you will create the VPC, add a subnet, attach a public gateway, and update a security group with rules to allow inbound traffic to the guestbook application. Start [here](vpc/vpcgen2.md).
1. **[Istio](https://ibm.github.io/istio101/)**, use Istio to manage network traffic, load balance across microservices, enforce access policies, verify service identity, and more.

This series of workshops on Kubernetes Networking is accompanied by a [lecture on Kubernetes Networking](https://raw.githubusercontent.com/IBM/kubernetes-networking/master/pdf/KubernetesNetworking-Lecture.pdf).

Kubernetes Networking is part of the series `Kubernetes Security`, which includes:

1. Kubernetes Security,
2. Kubernetes Networking,
3. Kubernetes Storage,
4. Kubernetes Automation (DevOps, IaC, CI/CD).

## Next steps

Continue your learning by visiting the [Istio workshop](https://ibm.github.io/istio101/). With Istio, you can manage network traffic, load balance across microservices, enforce access policies, verify service identity, and more.

## Prerequirementss

* Free IBM Cloud account, to create a new IBM Cloud account go [here](https://ibm.github.io/workshop-setup/NEWACCOUNT/).
* Free `Pay-As-You-Go` account. To upgrade a free IBM Cloud account, go [here](https://ibm.github.io/workshop-setup/PAYASYOUGO/).
* CognitiveLabs.ai account, to access a client terminal at CognitiveLabs.ai, go [here](https://ibm.github.io/workshop-setup/COGNITIVECLASS/).
* Kubernetes standard cluster v1.18 with at least 2 worker nodes on a VLAN, a subnet with public IPs, external LoadBalancer (for details about VLAN, subnets and IPs, see [here](https://cloud.ibm.com/docs/containers?topic=containers-subnets) ).

Go to [Setup](setup/setup1.md) for more details.

## Labs

1. **Lab1 Kubernetes Networking 101**
    1. [Setup](services/setup1.md)
    2. [Services](services/services.md)
    3. [ClusterIP](services/clusterip.md)
    4. [NodePort](services/nodeport.md)
    5. [Loadbalancer NLB](services/loadbalancer.md)
    6. [ExternalName](services/externalname.md)
    7. [Ingress ALB](services/ingress-alb.md)
1. **Lab2 Secured Routes**
    1. [Secured Routes and TLS Termination](route/secured-routes.md)
1. **Lab3 Network Policies**
    1. [Network Policy and Calico](calico/networkpolicy.md)
1. **Lab4 Secure a Cluster using VPC**
    1. [Setup](vpc/setup2.md)
    2. [VPC Gen2](vpc/vpcgen2.md)

## Technologies

This workshop was tested using the following technologies:

* IBM Cloud Kubernetes Service (IKS) version 1.19, 2 worker nodes, flavor u3c.2x4
* Calico client version v3.17.1
* Calico cluster version v3.16.5
* ibmcloud version 1.3.0
* ibmcloud container-service/kubernetes-service   1.0.28
* vpc-infrastructure/infrastructure-service 0.7.5
* kubectl version 1.19

## Contributors

* Remko de Knikker, [remkohdev](https://github.com/remkohdev),
* Masa Abushamleh, [nerdingitout](https://github.com/nerdingitout)
* Tim Robinson, [timroster](https://github.com/timroster),
