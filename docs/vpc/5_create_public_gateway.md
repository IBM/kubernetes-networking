# Create a Public Gateway

Create a public gateway, and update the subnet with the public gateway, this should create a floating IP for your public gateway attached to your subnet,

```bash
$ ibmcloud is public-gateway-create $MY_PUBLIC_GATEWAY $MY_VPC_ID $MY_ZONE

Creating public gateway bnewell-public-gateway1 in resource group bnewell-vpc-rg under account B. NEWELLs Account as user bnewell@email.com...

ID               r006-0cdf61b6-f0bc-455e-984e-999839f8c4cd
Name             bnewell-public-gateway1
CRN              crn:v1:bluemix:public:is:us-south-1:a/3fe3c0de197257ef62d81c9f9c0f33aa::public-gateway:r006-0cdf61b6-f0bc-455e-984e-999839f8c4cd   
Status           available
Zone             us-south-1
Created          2021-09-25T15:43:55+00:00
Floating IP      ID                                          Name                        Address      
                 r006-6ebbcbf0-98e1-441c-9599-fbbf3167231b   bnewell-public-gateway1   52.116.139.195      
                    
VPC              ID                                          Name      
                 r006-1b267b51-9651-4922-b95c-8a3243226207   bnewell-vpcgen2-vpc1      
                    
Resource group   ID                                 Name      
                 6542804b7f1040e59d7db25f53d5db18   bnewell-vpc-rg
```

Set an environment variable for the public gateway id,

```bash
MY_PUBLIC_GATEWAY_ID=$(ibmcloud is public-gateways --output json | jq -r '.[] | select( .name=='\"$MY_PUBLIC_GATEWAY\"') | .id')
echo $MY_PUBLIC_GATEWAY_ID
```

Configure the public gateway for the subnet,

```bash
$ ibmcloud is subnet-update $MY_VPC_SUBNET_ID --public-gateway-id $MY_PUBLIC_GATEWAY_ID

Updating subnet 0717-30195076-5cd6-4477-9fde-cd46e3fa03c7 under account B. NEWELLs Account as user bnewell@email.com...
                       
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
                       
Public Gateway      ID                                          Name      
                    r006-0cdf61b6-f0bc-455e-984e-999839f8c4cd   bnewell-public-gateway1      
                       
VPC                 ID                                          Name      
                    r006-1b267b51-9651-4922-b95c-8a3243226207   bnewell-vpcgen2-vpc1      
                       
Resource group      ID                                 Name      
                    6542804b7f1040e59d7db25f53d5db18   bnewell-vpc-rg
```

Retrieve the floating IP information for attached to the subnet,

```bash
MY_FLOATING_IP=$(ibmcloud is subnet-public-gateway $MY_VPC_SUBNET_ID --output json | jq -r '.floating_ip.address')
echo $MY_FLOATING_IP
```

Check that the public gateway is now attached to the subnet by retrieving the public gateway id from the subnet configuration,

```bash
MY_PUBLIC_GATEWAY_ID2=$(ibmcloud is subnets --output json | jq -r '.[] | select( .name=='\"$MY_VPC_SUBNET_NAME\"') | .public_gateway.id ')
echo $MY_PUBLIC_GATEWAY_ID2
```

## Next

Next, [Review VPC](6_review_vpc.md).
