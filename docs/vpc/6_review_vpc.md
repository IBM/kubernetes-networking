# Review VPC Infrastructure

To check your VPC resources in IBM Cloud, browse to the `VPC Infrastructure` dashboard at [https://cloud.ibm.com/vpc-ext/overview](https://cloud.ibm.com/vpc-ext/overview).

![VPC dashboard](../images/vpc/vpc-dashboard.png)

From the left menu, list the VPCs,

![VPCs](../images/vpc/vpcs.png)

List the [Subnets for VPC](https://cloud.ibm.com/vpc-ext/network/subnets),

![VPC Subnets](../images/vpc/subnets.png)

Click the linked subnet name to view the new subnet details,

![VPC Subnet Details](../images/vpc/subnet-details.png)

You should see a public gateway attached with a floating IP.

Go to [Floating IPs](https://cloud.ibm.com/vpc-ext/network/floatingIPs),

![Floating IPs for VPC](../images/vpc/floatingips.png)

Go to Public gateways,

![Public gateways for VPC](../images/vpc/publicgateways.png)

Go to Access control lists,

![Access control lists for VPC](../images/vpc/acls.png)

Notice, there is 1 `Inbound rule` and 1 `Outbound rule` defined.

Go to Security groups,

![Security groups for VPC](../images/vpc/security-groups.png)

Click on the linked security group to review the secuirty group's details,

![Security group details](../images/vpc/security-group-details.png)

Click the `Rules` tab,

![Security group Rules](../images/vpc/security-group-rules.png)

## Conclusion

You are awesome! You created a Virtual Private Cloud (VPC) with a subnet, a security group with access rules to control inbound and outbound traffic, and attached a public gateway with a floating IP as an access point.

## Next

Next, [create a Red Hat OpenShift Kubernetes Service (ROKS)](../vpc_roks/1_create_roks_for_vpc.md) or create an IBM Cloud Kubernetes Service instance.
