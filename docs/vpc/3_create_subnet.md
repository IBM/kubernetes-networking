# Create a Subnet

Create a subnet

```console
$ ibmcloud is subnet-create $MY_VPC_SUBNET_NAME $MY_VPC_ID --zone $MY_ZONE --ipv4-address-count 256

Creating subnet bnewell-vpcsubnet1 in resource group bnewell-vpc-rg under account B. NEWELL's Account as user bnewell@email.com...

ID                  0717-30195076-5cd6-4477-9fde-cd46e3fa03c7   
Name                bnewell-vpcsubnet1   
CRN                 crn:v1:bluemix:public:is:us-south-1:a/3fe3c0de197257ef62d81c9f9c0f33aa::subnet:0717-30195076-5cd6-4477-9fde-cd46e3fa03c7   
Status              pending   
IPv4 CIDR           10.240.0.0/24   
Address available   251   
Address total       256   
Zone                us-south-1   
Created             2021-09-25T15:30:18+00:00   
ACL                 ID                                          Name      
                    r006-d3ecc69b-57e5-4618-992c-24a1ffc99976   stint-gainfully-delicacy-perceive      
                       
Routing table       ID                                          Name      
                    r006-85df0100-0c86-4f1a-b182-9eb692740f2c   reconvene-stoplight-freebie-preamble      
                       
Public Gateway      -   
VPC                 ID                                          Name      
                    r006-1b267b51-9651-4922-b95c-8a3243226207   bnewell-vpcgen2-vpc1      
                       
Resource group      ID                                 Name      
                    6542804b7f1040e59d7db25f53d5db18   bnewell-vpc-rg
```

Set an environment variable `MY_VPC_SUBNET_ID`,

```console
MY_VPC_SUBNET_ID=$(ibmcloud is subnets --output json | jq -r '.[] | select( .name=='\"$MY_VPC_SUBNET_NAME\"') | .id ')
echo $MY_VPC_SUBNET_ID
```

## Next

Next, [Review the Security Group](4_review_security_group.md).
