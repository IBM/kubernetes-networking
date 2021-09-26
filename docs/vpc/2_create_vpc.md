# Create a Virtual Private Cloud (VPC)

In this section, you will create and configure a Virtual Private Cloud (VPC) and configure the related resources for your VPC like the subnet, security group and floating IP.

## Environment Variables

Set the following environment variables to be used in the remainder of the tutorial. Set the USERNAME to a unique value shorter than 10 characters and set the IBMID to the email you used to create the IBM Cloud account. The other variables don't need to be changed but you can choose to rename them if you prefer.

```bash
USERNAME=<bnewell>
IBMID=<b.newell2@email.com>
```

Using the previous values, set the following environment variables,

```bash
RESOURCE_GROUP=$USERNAME-vpc-rg
MY_REGION=us-south
MY_ZONE=$MY_REGION-1
MY_VPC_NAME=$USERNAME-vpc1
MY_VPC_SUBNET_NAME=$USERNAME-vpcsubnet1
MY_PUBLIC_GATEWAY=$USERNAME-public-gateway1
MY_NAMESPACE=my-guestbook
```

## Create Resource Group

```bash
$ ibmcloud resource group-create $RESOURCE_GROUP

Creating resource group remkohdev-vpc-rg under account 3fe3c0de197257ef62d81c9f9c0f33aa as b.newell2@email.com...
OK
Resource group bnewell-vpc-rg was created.
Resource Group ID: 6542804b7f1040e59d7db25f53d5db18
```

## Set Targets

Target the region and the resource group to create our VPC environment.

```bash
ibmcloud target -r $MY_REGION -g $RESOURCE_GROUP
```

The VPC must be set up in the same multi-zone metro location where you want to create your cluster. Target generation 2 infrastructure.

```bash
$ ibmcloud is target --gen 2
Target Generation: 2
```

## Create a VPC

This section uses the IBM Cloud CLI to create a cluster in your VPC on generation 2 compute.

Create a VPC,

```console
$ ibmcloud is vpc-create $MY_VPC_NAME

Creating vpc bnewell-vpcgen2-vpc1 in resource group bnewell-vpc-rg under account B. Newell's Account as user bnewell@email.com...
                                                  
ID                                             r006-1b267b51-9651-4922-b95c-8a3243226207   
Name                                           bnewell-vpcgen2-vpc1   
CRN                                            crn:v1:bluemix:public:is:us-south:a/3fe3c0de197257ef62d81c9f9c0f33aa::vpc:r006-1b267b51-9651-4922-b95c-8a3243226207   
Status                                         pending   
Classic access                                 false   
Created                                        2021-09-25T15:26:39+00:00   
Resource group                                 ID                                 Name      
                                               6542804b7f1040e59d7db25f53d5db18   bnewell-vpc-rg      
                                                  
Default network ACL                            ID                                          Name      
                                               r006-d3ecc69b-57e5-4618-992c-24a1ffc99976   stint-gainfully-delicacy-perceive      
                                                  
Default security group                         ID                                          Name      
                                               r006-ba25832d-43da-4be4-a17f-661512eac237   empathy-levitator-flogging-gleeful      
                                                  
Default routing table                          ID                                          Name      
                                               r006-85df0100-0c86-4f1a-b182-9eb692740f2c   reconvene-stoplight-freebie-preamble      
                                                  
                                                  
Cloud Service Endpoint source IP addresses:    Zone         Address      
                                               us-south-1   10.249.195.37      
                                               us-south-2   10.12.157.227      
                                               us-south-3   10.16.248.27 
```

Check your VPC was created successfully,

```bash
$ ibmcloud is vpcs

Listing vpcs in resource group bnewell-vpc-rg and region us-south under account B. NEWELL's Account as user bnewell@email.com...
ID                                          Name                     Status      Classic access   Default network ACL                 Default security group               Resource group   
r006-1b267b51-9651-4922-b95c-8a3243226207   bnewell-vpcgen2-vpc1   available   false            stint-gainfully-delicacy-perceive   empathy-levitator-flogging-gleeful   bnewell-vpc-rg  
```

Set an environment variable `MY_VPC_ID`,

```bash
MY_VPC_ID=$(ibmcloud is vpcs --output json | jq -r '.[] | select( .name=='\"$MY_VPC_NAME\"') | .id ')
echo $MY_VPC_ID
```

## Next

Next, [Create a Subnet](3_create_subnet.md).
