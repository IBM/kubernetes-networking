# About this Workshop

## Prerequisites

* IBM Cloud Pay-as-You-Go account,
* IBM Cloud Shell,
* VPC infrastructure service plug-in,

## About

This tutorial explains how to start to `air-gap` and secure an OpenShift or Kubernetes cluster and physically isolate your cluster using a Virtual Private Cloud (VPC). Using an air gapped cluster is one of the first things you will do to secure your container deployments. For more information about `air-gap`, go [here](airgap.md).

The first step to secure your cluster is to create a Virtual Private Cloud (VPC) and add rules to a `Security Group` of an Application Load Balancer (ALB) to allow certain inbound traffic. You can add a gateway like `API Connect` to the VPC and expose the gateway to manage traffic, for instance by using OAuth, rate limiting and API key access control.

With Red Hat OpenShift Kubernetes (ROKS) or IBM Cloud Kubernetes Service (IKS) on VPC Generation 2 Computeon IBM Cloud, you can create an OpenShift or Kubernetes cluster in Virtual Private Cloud (VPC) infrastructure in a matter of minutes. OpenShift is a more enterprise-ready secure Container Orchestration (CO) platform, but in this tutorial I use a plain managed Kubernetes service because it is more accessible and affordable. If you want however, you can choose to use OpenShift instead.

This tutorial uses a free IBM Cloud account, a free Pay-As-You-Go account, a free IBM Cloud Kubernetes Service (IKS) service with Kubernetes v1.18 with 1 worker node, and a free Virtual Private Cloud (VPC) service.

This tutorial is based on the official documentation for [creating a cluster in your Virtual Private Cloud (VPC) on generation 2 compute](https://cloud.ibm.com/docs/containers?topic=containers-vpc_ks_tutorial).

Steps:

1. Setup,
1. Create a VPC Generation 2 Compute using CLI
1. Update the Security Group
1. Understanding the Load Balancer
1. Ingress Application Load Balancer (ALB)
1. Cleanup
1. [Optional] Using UI to Create a Cluster with VPC Generation 2 Compute

## Setup Kubernetes Cluster

For this tutorial you need access to a Kubernetes or OpenShift cluster on IBM Cloud.

* To use an IBM provided Kubernetes cluster with your IBM Cloud account and IBMId, grant access permissions to the cluster, as instructed [here](https://ibm.github.io/workshop-setup/GRANTCLUSTER/).
* To connect to a managed `Red Hat OpenShift Kubernetes Service (ROKS)`, go [here](https://ibm.github.io/workshop-setup/ROKS/).

## Connect to Cluster

1. Optionally, if you don't know your cluster name, list all clusters, and search for your cluster by name,

    ```console
    KS_NAME_SUB=<substring>
    ibmcloud ks clusters --output json | jq -r 'map(select(.name | contains('\"$KS_NAME_SUB\"')))'
    ```

2. Set the `KS_CLUSTER_NAME` environment variable to the correct cluster name, and the namespace environment variable `$MY_NS` to the desired namespace,

    ```console
    KS_CLUSTER_NAME=<your cluster name>
    MY_NS=my-apps
    ```

3. For IKS, download the cluster configuration to the client,

    ```console
    ibmcloud ks cluster config --cluster $KS_CLUSTER_NAME
    kubectl config current-context
    ```

4. The config should be set to a `clustername/clusterid` pair,
5. For OpenShift, find the `oc login` command with the login token. To connect to a managed `Red Hat OpenShift Kubernetes Service (ROKS)`, and download the session configuration, follow the instructions [here](https://ibm.github.io/workshop-setup/ROKS/). For example,

    ```console
    oc login --token=e6PfXsQKjxiUf7qkb4jJxEd851pa0ZuUSEimVuGt4aQ --server=https://c114-e.us-south.containers.cloud.ibm.com:32115
    ```

## Next

Next, [Create a VPC](2_create_vpc.md).
