# Airgap

An [`air gap`](https://en.wikipedia.org/wiki/Air_gap_(networking)), `air wall`, `air gapping` or `disconnected network` is a network security measure to ensure that a secure computer network is physically isolated from unsecured networks, such as the public Internet or an unsecured local area network. It means a computer or network has no network interfaces connected to other networks.

For most enterprise clients in regulated industries and also public sector agencies, the term `air gap` has more specific meaning. It's actually explained in the 2nd paragraph of the wikipedia link in the opening sentence of the blog post:

"To move data between the outside world and the air-gapped system, it is necessary to write data to a physical medium such as a thumbdrive, and physically move it between computers."

In an air-gapped Kubernetes cluster, you would not be able to deploy a Docker Hub image like ibmcom/guestbook:v1 . Instead, an air-gapped Kubernetes cluster would rely on a private registry. In theory, a container runtime can be configured with an image mirror to a private registry (thereby telling the container runtime to retrieve docker.io/ibmcom/guestbook:v1 from somewhere else, but it doesn't seem like this article is explaining that.

Also in an air-gapped Kubernetes cluster, one would not expect the control plane endpoint to be reachable over over the Internet. As created with the:

```
ibmcloud ks cluster create vpc-gen2 --name $MY_CLUSTER_NAME --zone $MY_ZONE --version $KS_VERSION --flavor bx2.2x8 --workers 1 --vpc-id $MY_VPC_ID --subnet-id $MY_VPC_SUBNET_ID
```

command, this cluster will have public service endpoints. To disable the creation of these, use the --disable-public-service-endpoint option for the create.

So - what are some things that are done by enterprises right out of the gate when using a public cloud provider's Kubernetes service to create isolation (note I'm not saying "air-gap") that are best practices?

1. Disable the public gateway for the VPC subnet with the workers. This means that Internet access won't occur from the workers, including pulling images from Docker Hub. How do workers in such clusters pull container images for their workloads? IBM Container Registry is a great option. Organizations can have their CI systems tag and push images to IBM CR and then k8s/openshift clusters that don't have Internet access can pull images from IBM CR.
2. Disable public endpoints for the control plane. When using private only endpoints, orgs can use systems within the VPC to manage the control plane, or set up Direct Link / VPN connections to their VPC. There presently are some limitations to how control plane access works through private service endpoints, but work is in the roadmap for supporting access to control plane endpoints through these connections. In the interim, proxying these connections through an haproxy instance in the VPC looks to be an ok way to go.