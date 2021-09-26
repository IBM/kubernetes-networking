# Cleanup

To conclude, you can choose to delete your Kubernetes resources for this tutorial.

During the cleanup, you need the following environment variables,

USERNAME
MY_CLUSTER_NAME
SVC_NODEPORT
MY_DEFAULT_SG_ID
MY_DEFAULT_SG_RULE_ID
MY_VPC_SUBNET_ID
PUBLIC_GATEWAY_ID
MY_VPC_NAME

## Delete Cluster

```bash
$ ibmcloud ks cluster rm --cluster $MY_CLUSTER_NAME

Do you want to delete the persistent storage for this cluster? If yes, the data cannot be recovered. If no, you can delete the persistent storage later in your IBM Cloud infrastructure account. [y/N]> y
After you run this command, the cluster cannot be restored. Remove the cluster remkohdev-iks118-vpc-cluster1? [y/N]> y
Removing cluster remkohdev-iks118-vpc-cluster1, persistent storage...
OK
```

Now the Kubernetes cluster is deleted, we need to first remove the public gateway and load balancers from the subnet, then remove the subnet from the VPC, and delete the VPC.

Delete the Gateways, Load Balancers, Network Interfaces, subnet, public gateways, and finally delete the VPC,

## Delete Security Group Rules

Find the Security Group id,

```bash
ibmcloud is security-groups
```

<!--
ibmcloud is security-group-rules $MY_DEFAULT_SG_ID [--output json]
MY_DEFAULT_SG_RULE_ID=$(ibmcloud is security-group-rules $MY_DEFAULT_SG_ID --output json | jq -r --arg SVC_NODEPORT $SVC_NODEPORT  '.[] | select( .port_max==($SVC_NODEPORT|tonumber)) | .id ')
echo $MY_DEFAULT_SG_RULE_ID-->

Repeat to delete all rules for the security group,

```bash
$ ibmcloud is security-group-rule-delete $MY_DEFAULT_SG_ID $MY_DEFAULT_SG_RULE_ID

This will delete security group rule r006-4f5f795f-fa90-44e4-b9c4-cb9457a2a421 and cannot be undone. Continue [y/N] ?> y
Deleting rule r006-4f5f795f-fa90-44e4-b9c4-cb9457a2a421 from security group  r006-db96d593-c224-497d-888c-03d84f6d8e98 under account Remko de Knikker as user b.newell2@remkoh.dev...
OK
Rule r006-4f5f795f-fa90-44e4-b9c4-cb9457a2a421 is deleted.
```

## Delete Public Gateway

Find the subnet id,

```bash
ibmcloud is subnets

MY_VPC_SUBNET_ID=<your subnet id>
```

Detach the public gateway from the subnet,

```bash
$ ibmcloud is subnet-public-gateway-detach $MY_VPC_SUBNET_ID

Detaching public gateway from subnet 0717-57ebaf2d-0de6-4630-af01-6cd84031b679 under account B. Newell as user b.newell2@remkoh.dev...
OK
Public gateway is detached.
```

Delete the public gateway,

```bash
ibmcloud is public-gateways
PUBLIC_GATEWAY_ID=$(ibmcloud is public-gateways --output json | jq '.[0]' | jq -r '.id' ) 
echo $PUBLIC_GATEWAY_ID

$ ibmcloud is public-gateway-delete $PUBLIC_GATEWAY_ID

This will delete public gateway r006-f4603b78-839b-42a5-949c-76403948821a and cannot be undone. Continue [y/N] ?> y
Deleting public gateway r006-f4603b78-839b-42a5-949c-76403948821a under account B. Newell as user b.newell2@remkoh.dev...
OK
Public gateway r006-f4603b78-839b-42a5-949c-76403948821a is deleted.
```

## Delete the Subnet

Delete the subnet,

```bash
$ ibmcloud is subnet-delete $MY_VPC_SUBNET_ID

This will delete Subnet 0717-57ebaf2d-0de6-4630-af01-6cd84031b679 and cannot be undone. Continue [y/N] ?> y
Deleting subnet 0717-57ebaf2d-0de6-4630-af01-6cd84031b679 under account B. Newell as user b.newell2@remkoh.dev...
OK
Subnet 0717-57ebaf2d-0de6-4630-af01-6cd84031b679 is deleted.
```

## Delete the VPC

Find all VPCs,

```bash
ibmcloud is vpcs

MY_VPC_NAME=<your VPC name>
```

Delete the Virtual Private Cloud,

```bash
MY_VPC_ID=$(ibmcloud is vpcs --output json | jq -r '.[] | select( .name=='\"$MY_VPC_NAME\"') | .id ')
echo $MY_VPC_ID

$ ibmcloud is vpc-delete $MY_VPC_ID

This will delete vpc r006-3c9ab19c-e8af-4eb3-ab60-b9777c3cce1c and cannot be undone. Continue [y/N] ?> y
Deleting vpc r006-3c9ab19c-e8af-4eb3-ab60-b9777c3cce1c under account Remko de Knikker as user b.newell2@remkoh.dev...
OK
vpc r006-3c9ab19c-e8af-4eb3-ab60-b9777c3cce1c is deleted.
```

## Delete the Resource Group

Find resource groups,

```bash
ibmcloud resource groups
```

```bash
$ ibmcloud is resource-group-delete $USERNAME-vpc-rg

Really delete the resource group bnewell-vpc-rg? [y/N]> y
Deleting resource group bnewell-vpc-rg under account 3fe3c0de197257ef62d81c9f9c0f33aa as bnewell@email.com...
OK
Resource group bnewell-vpc-rg was deleted successfully
```

## Conclusion

Thank you for cleaning up!
