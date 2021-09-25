# Setup

In the previous lab, we defined environment variables for the region and zone, and created a VPC and subnet. Start to check that these required variables are valid and available,

```bash
$ echo $MY_ZONE
us-east-1

echo $MY_VPC_ID
echo $MY_VPC_SUBNET_ID
```

To list available VPCs and Subnets,

```bash
ibmcloud is vpcs
ibmcloud is subnets
```

To set an environment variable `MY_VPC_ID` using `MY_VPC_NAME`,

```bash
MY_VPC_ID=$(ibmcloud is vpcs --output json | jq -r '.[] | select( .name=='\"$MY_VPC_NAME\"') | .id ')
echo $MY_VPC_ID
```

To set an environment variable `MY_VPC_SUBNET_ID` using `MY_VPC_SUBNET_NAME`,

```bash
MY_VPC_SUBNET_ID=$(ibmcloud is subnets --output json | jq -r '.[] | select( .name=='\"$MY_VPC_SUBNET_NAME\"') | .id ')
echo $MY_VPC_SUBNET_ID
```

To check the available zones,

```bash
ibmcloud ks zone ls --provider vpc-gen2
```

Check the available Kubernetes service versions,

```bash
ibmcloud ks versions
```

Set an environment variable for the Kubernetes version using the default version for Kubernetes,

```bash
export KS_VERSION=1.20.10
```

You need to have set environment variables for `KS_VERSION` and `MY_ZONE`, `MY_VPC_ID` and `MY_VPC_SUBNET_ID`, the latter 3 were created in [Lab 4 Create a VPC](../vpc/2_create_vpc.md).

## Next

Go to [Create an IBM Cloud Kubernetes Service instance for VPC](2_create_iks_for_vpc.md).
