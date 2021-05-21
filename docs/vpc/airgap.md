# What is Airgap

An [`air gap`](https://en.wikipedia.org/wiki/Air_gap_(networking)), `air wall`, `air gapping` or `disconnected network` is a network security measure to ensure that a secure computer network is physically isolated from unsecured networks, such as the public Internet or an unsecured local area network. It means a computer or network has no network interfaces connected to other networks.

For most enterprise clients in regulated industries and public sector agencies, the term `air gap` has a more specific meaning.

## Storage

"To move data between the outside world and the air-gapped system, it is necessary to write data to a physical medium such as a thumbdrive, and physically move it between computers."
[source](https://en.wikipedia.org/wiki/Air_gap_(networking))

## Container Registry

In an air-gapped Kubernetes cluster, you would not be able to deploy a Docker Hub image like ibmcom/guestbook:v1 . Instead, an air-gapped Kubernetes cluster would rely on a private registry. In theory, a container runtime can be configured with an image mirror to a private registry (thereby telling the container runtime to retrieve docker.io/ibmcom/guestbook:v1 from somewhere else, but it doesn't seem like this article is explaining that.

## Public Service Endpoint

In a properly air-gapped Kubernetes cluster, you are not able to reach the control plane endpoint from outside the airgapped network, e.g. over the Internet.

When you create an IBM Cloud Kubernetes Service with VPC instance, as in:

```bash
ibmcloud ks cluster create vpc-gen2 --name $MY_CLUSTER_NAME --zone $MY_ZONE --version $KS_VERSION --flavor bx2.2x8 --workers 1 --vpc-id $MY_VPC_ID --subnet-id $MY_VPC_SUBNET_ID
```

the created cluster will have public service endpoints. To disable the creation of these, use the `--disable-public-service-endpoint` flag.

## Best Practices

1. Disable the public gateway for the VPC subnet with the worker nodes. This means that you cannot access the cluster nodes from outside the secure network, which includes for instance a build server and a container registry.

2. Disable public endpoints for the control plane. When using private only endpoints, orgs can use systems within the VPC to manage the control plane, or set up `Direct Link / VPN` connections to their VPC. If this is not possible, proxying connections through a `haproxy` instance in the VPC is an alternative.

3. How can a container engine on a worker in such a cluster pull container images to create containers? One option is to use a private registry inside the network. IBM Container Registry, the built-in OpenShift private registry, or a custom registry deployment inside the network are great options. Organizations can have their Continuous Integration (CI) systems tag and push images to IBM CR and then k8s/openshift clusters that don't have Internet access can pull images from IBM CR.

4. Traffic to RESTful API endpoints on your cluster can be proxied through a secure gateway like API Connect with DataPower gateway.
