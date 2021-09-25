# Kubernetes Networking

This series of workshops on Kubernetes Networking is accompanied by a [lecture on Kubernetes Networking](https://raw.githubusercontent.com/IBM/kubernetes-networking/master/pdf/KubernetesNetworking-Lecture.pdf).

The Kubernetes Networking series consists of the following topics:

1. **Kubernetes Networking 101** (60 mins), you will use different ways to control traffic on a Kubernetes cluster with Service types. Start [here](services/services.md).
1. **Add an Ingress on OpenShift** (15 minutes), add an Ingress and Route to expose a Service, you will use different types of TLS termination to secure Routes on OpenShift: edge, passthrough and reencrypt. Start [here](route/route.md).
1. **Network Policies and Calico** (15 minutes), create a Network Policy and use Calico. Start [here](calico/networkpolicy.md).
1. **Create a Virtual Private Cloud (VPC)** (90 mins), you will create the VPC, add a subnet, attach a public gateway, and review the security group that allows inbound and outbound access. Start [here](vpc/0_about.md).
1. **Create a Kubernetes Cluster for VPC** you will create a IBM Cloud Kubernetes Service (IKS) for VPC and deploy a guestbook application to a Kubernetes cluster in VPC, and update a security group with rules to allow inbound traffic to the guestbook application. Start [here](vpc_iks/0_about.md).
1. **[Istio](https://ibm.github.io/istio101/)**, use Istio to manage network traffic, load balance across microservices, enforce access policies, verify service identity, and more.

## Labs

1. **Lab1 Kubernetes Networking 101**
    1. [Setup](services/setup1.md)
    2. [Services](services/services.md)
    3. [ClusterIP](services/clusterip.md)
    4. [NodePort](services/nodeport.md)
    5. [Loadbalancer NLB](services/loadbalancer.md)
    6. [ExternalName](services/externalname.md)
    7. [Ingress ALB](services/ingress-alb.md)
1. **Lab2 Ingress**
    1. [Ingress and ALB](ingress/ingress-alb.md),
    1. [Route](route/route.md),
    1. [Secured Routes and TLS Termination](route/secured-routes.md),
1. **Lab3 Network Policies**
    1. [Network Policy and Calico](calico/networkpolicy.md)
1. **Lab4 Create a VPC**
    1. [Setup](vpc/0_setup.md),
    1. [About](vpc/1_about.md),
    1. [Create a VPC](vpc/2_create_vpc.md),
    1. [Create a Subnet](vpc/3_create_subnet.md),
    1. [Review the Security Group](vpc/4_review_security_group.md),
    1. [Create a Public Gateway with Floating IP](vpc/5_create_public_gateway.md),
    1. [Review the VPC](vpc/6_review_vpc.md).
1. **Lab5 Create a Kubernetes Cluster for VPC**
    1. [Setup](vpc_iks/1_setup.md),
    1. [Create an IBM Cloud Kubernetes Service (IKS) for VPC](vpc_iks/2_create_iks_for_vpc.md),

## Related Workshops

The following series are related to `Kubernetes Networking`:

1. Kubernetes Security,
2. Kubernetes Networking,
3. Kubernetes Storage,
4. Kubernetes Automation (Secure DevOps, IaC, CI/CD).

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
