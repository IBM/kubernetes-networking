## [Optional] Using UI to Create a Cluster with VPC Generation 2 Compute

This section uses the browser to create the cluster and VPC. I always prefer the command line or CLI instead because this forces me to write the commands that allow me to automate the whole process and to think of everything as code. If you want to use the CLI, skip to the next section.

Using the UI, create an IBM Cloud Kubernetes Service (IKS), for infrastructure provider choose Generation 2 Compute instead of Classic. For a comparison of infrastructure providers, read Supported Infrastructure Providers, or read an Overview of VPC Networking in IBM Cloud Kubernetes Service (IKS).

This will setup the following infrastructure for you:

1. Virtual Private Cloud, watch this good [primer on VPC by Ryan Sumner](https://www.ibm.com/cloud/blog/virtual-private-cloud),
2. Subnet,
3. Security Group,
4. Access Control List (ACL),
5. A Public Gateway with a Floating IP for Public Access,
6. An encrypted VPN Gateway or a Direct Link Private Circuits for Private Access,
7. An elastic Load Balancer for VPC, using an Application Load Balancer (ALB), which includes Sysdig monitoring, Hihg Availability (HA) with a Domain Name Server (DNS), Multi-Zone Region (MZR) support, L4 network layer and L7 application layer support, both public and private load balancing.
