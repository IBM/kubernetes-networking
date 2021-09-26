# Review Security Group

Inspect the VPC's default security group and set the environment variables `MY_DEFAULT_SG_NAME` and `MY_DEFAULT_SG_ID`,

```console
MY_DEFAULT_SG_NAME=$(ibmcloud is vpcs --output json | jq -r '.[] | select( .name=='\"$MY_VPC_NAME\"') | .default_security_group.name ')
echo $MY_DEFAULT_SG_NAME

MY_DEFAULT_SG_ID=$(ibmcloud is security-groups --output json | jq -r '.[] | select( .name=='\"$MY_DEFAULT_SG_NAME\"') | .id ')
echo $MY_DEFAULT_SG_ID

ibmcloud is security-group $MY_DEFAULT_SG_ID --output json

{
    "created_at": "2021-09-25T15:26:39.000Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/3fe3c0de197257ef62d81c9f9c0f33aa::security-group:r006-ba25832d-43da-4be4-a17f-661512eac237",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/security_groups/r006-ba25832d-43da-4be4-a17f-661512eac237",
    "id": "r006-ba25832d-43da-4be4-a17f-661512eac237",
    "name": "empathy-levitator-flogging-gleeful",
    "network_interfaces": [],
    "resource_group": {
        "href": "https://resource-controller.cloud.ibm.com/v2/resource_groups/6542804b7f1040e59d7db25f53d5db18",
        "id": "6542804b7f1040e59d7db25f53d5db18",
        "name": "bnewell-vpc-rg"
    },
    "rules": [
        {
            "direction": "outbound",
            "id": "r006-adaee65e-41ab-4307-8e7b-51f4b8a454d9",
            "ip_version": "ipv4",
            "protocol": "all",
            "remote": {
                "cidr_block": "0.0.0.0/0"
            }
        },
        {
            "direction": "inbound",
            "id": "r006-690d7205-55c0-458b-8747-0b4c43b2b1a6",
            "ip_version": "ipv4",
            "protocol": "all",
            "remote": {
                "href": "https://us-south.iaas.cloud.ibm.com/v1/security_groups/r006-ba25832d-43da-4be4-a17f-661512eac237",
                "id": "r006-ba25832d-43da-4be4-a17f-661512eac237",
                "name": "empathy-levitator-flogging-gleeful"
            }
        }
    ],
    "targets": [],
    "vpc": {
        "crn": "crn:v1:bluemix:public:is:us-south:a/3fe3c0de197257ef62d81c9f9c0f33aa::vpc:r006-1b267b51-9651-4922-b95c-8a3243226207",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpcs/r006-1b267b51-9651-4922-b95c-8a3243226207",
        "id": "r006-1b267b51-9651-4922-b95c-8a3243226207",
        "name": "bnewell-vpcgen2-vpc1"
    }
}
```

## Next

Next, [Create a Public Gateway](5_create_public_gateway.md).
