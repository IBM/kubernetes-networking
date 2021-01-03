# Setup

## Prerequirements

* Free IBM Cloud account, to create a new IBM Cloud account go [here](https://ibm.github.io/workshop-setup/NEWACCOUNT/).
* Free `Pay-As-You-Go` account. To upgrade a free IBM Cloud account, go [here](https://ibm.github.io/workshop-setup/PAYASYOUGO/).
* CognitiveLabs.ai account, to access a client terminal at CognitiveLabs.ai, go [here](https://ibm.github.io/workshop-setup/COGNITIVECLASS/).
* Kubernetes cluster v1.18:
    - for labs in `Services`, `ClusterIP`, and `NodePort`, you need a Kubernetes cluster with at least 1 worker node.
    - for labs in  `LoadBalancer` and `Ingress` you need a Kubernetes cluster with at least 2 worker nodes.
    - for labs in `Network Policy` and `Calico`, you need a Kubernetes cluster with at least 2 worker nodes.
    - for labs in `VPC Gen2`, you need a Kubernetes cluster with at least 1 worker node.

## Setup Kubernetes Cluster

* To create a free IBM Cloud Kubernetes Service (IKS) with 1 worker node, you can upgrade the free IBM Cloud account to a free `Pay-As-You-Go` account. To upgrade, go [here](https://ibm.github.io/workshop-setup/PAYASYOUGO/).
* To create a free IBM Cloud Kubernetes Service (IKS) cluster with 1 worker node, 
* To grant permissions to an existing Kubernetes cluster, as part of an IBM managed workshop, you can access an existing cluster using your IBM Cloud account and IBM Id. To grant access permissions to a cluster, go [here](https://ibm.github.io/workshop-setup/GRANTCLUSTER/).
* To connect to a managed `Red Hat OpenShift Kubernetes Service (ROKS)`, go [here](https://ibm.github.io/workshop-setup/ROKS/).

## Setup Client Terminal

I recommended using the browser based client terminal that is party of the Labs environment at CognitiveLabs. to access a client terminal at CognitiveLabs.ai, go [here](https://ibm.github.io/workshop-setup/COGNITIVECLASS/). 

Alternatively, you can also use the IBM Cloud shell with tools pre-installed to run the labs at https://shell.cloud.ibm.com.

The client terminal needs to have `Docker`, `kubectl`, for OpenShift you need the `oc` CLI, the `IBM Cloud CLI` and the `IBM Cloud Kubernetes Service plug-in`.

## Install Calicoctl

Install calicoctl,

```
$ curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/v3.17.1/calicoctl
$ chmod +x calicoctl
$ echo 'export PATH=$HOME:$PATH' > .bash_profile
$ source .bash_profile
$ ibmcloud ks cluster config --cluster $KS_CLUSTER_NAME --admin --network
```

With Docker container,

```
docker pull calico/ctl:v3.17.1
```

As kubectl plugin,

```
curl -o kubectl-calico -L  https://github.com/projectcalico/calicoctl/releases/download/v3.17.1/calicoctl
chmod +x kubectl-calico
kubectl calico -h
```

## Login to IBM Cloud

1. Log in to your cluster, e.g. if created in the `us-south` region,

    ```
    IBMID=<your IBMID email>
    ibmcloud login -u $IBMID
    ```

1. If you are using federated SSO login, use the `-sso` flag instead.
1. Select the account in which the cluster was created.

    ![Login to IBM Cloud](images/shell-login-to-cloud.png)

2. Optionally, list all clusters and set the `KS_CLUSTER_NAME` environment variable to the correct cluster name,

    ```
    ibmcloud ks clusters
    KS_CLUSTER_NAME=<your cluster name>
    MY_NS=my-apps
    ```

3. Download the cluster configuration to the client,

    ```
    ibmcloud ks cluster config --cluster $KS_CLUSTER_NAME
    kubectl config current-context
    ```

4. The config should be set to a `clustername/clusterid` pair,

## HelloWorld App

1. From the Cloud shell, clone the guestbook application,

    ```
    $ git clone https://github.com/remkohdev/helloworld.git
    $ ls -al
    ```
