# About this Workshop

## Prerequisites

* IBM Cloud Pay-as-You-Go account,
* IBM Cloud Shell,
* VPC infrastructure service plug-in,

## About

This tutorial explains how to start to `air-gap` and secure an OpenShift or Kubernetes cluster and physically isolate your cluster using a Virtual Private Cloud (VPC). Using an air gapped cluster is one of the first things you will do to secure your container deployments. For more information about `air-gap`, go [here](airgap.md).

The first step to secure your cluster is to create a Virtual Private Cloud (VPC) and add rules to a `Security Group` of an Application Load Balancer (ALB) to allow certain inbound traffic. You can add a gateway like `API Connect` to the VPC and expose the gateway to manage traffic, for instance by using OAuth, rate limiting and API key access control.

With Red Hat OpenShift Kubernetes (ROKS) or IBM Cloud Kubernetes Service (IKS) on VPC Generation 2 Computeon IBM Cloud, you can create an OpenShift or Kubernetes cluster in Virtual Private Cloud (VPC) infrastructure in a matter of minutes. OpenShift is a more enterprise-ready secure Container Orchestration (CO) platform, but in this tutorial I use a plain managed Kubernetes service because it is more accessible and affordable. If you want however, you can choose to use OpenShift instead.

This tutorial uses a free IBM Cloud account, a free Pay-As-You-Go account, the IBM Cloud Shell, and a free Virtual Private Cloud (VPC) service.

This tutorial is based on the official documentation for [Using the IBM Cloud console to create VPC resources](https://cloud.ibm.com/docs/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console). Read more [About networking for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-about-networking-for-vpc).

Steps:

1. [Setup](0_setup.md),
1. [Create a VPC](1_create_vpc.md),
1. [Create a Subnet](3_create_subnet.md),
1. [Review the Security Group](4_review_security_group.md),
1. [Create a Public Gateway with Floating IP](5_create_public_gateway.md),
1. [Review the VPC](6_review_vpc.md).

## Next

Next, [Create a VPC](2_create_vpc.md).
