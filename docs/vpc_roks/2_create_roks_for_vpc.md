# Create a Red Hat Openshift Kubernetes Service (ROKS) Cluster

Create a cluster in your VPC in the same zone as the subnet. By default, your cluster is created with a public and a private service endpoint. You can use the public service endpoint to access the Kubernetes master,

We already defined environment variables for the region, zone and Kubernetes version, but start to check these are valid and available,

```console
ibmcloud ks zone ls --provider vpc-gen2
ibmcloud ks versions
```

Create the cluster in the VPC,

```console
$ ibmcloud ks cluster create vpc-gen2 --name $MY_CLUSTER_NAME --zone $MY_ZONE --version $KS_VERSION --flavor bx2.2x8 --workers 1 --vpc-id $MY_VPC_ID --subnet-id $MY_VPC_SUBNET_ID

Creating cluster...
OK
Cluster created with ID bvglln7d0e5j0u9lfa80
```

Review your IBM Cloud account resources,

Click the linked cluster name of the cluster you just created. If you do not see the cluster listed yet, wait and refresh the page. Check the status of the new cluster,

```bash
ibmcloud ks clusters

MY_CLUSTER_ID=$(ibmcloud ks clusters --output json --provider vpc-gen2 | jq -r '.[] | select( .name=='\"$MY_CLUSTER_NAME\"') | .id ')
echo $MY_CLUSTER_ID
```

To continue with the next step, the cluster status and the Ingress status must indicate to be available. The cluster might take about 15 minutes to complete.

```bash
ibmcloud ks cluster get --cluster $MY_CLUSTER_ID
```

![IBM Cloud Kubernetes Service Overview](images/ibmcloud-iks-overview.png)

Once the cluster is fully provisioned including the Ingress Application Load Balancer (ALB), you should see the `Worker nodes` status to be `100% Normal`, the `Ingress subdomain` should be populated, among other,

![IBM Cloud Kubernetes Service Overview](images/ibmcloud-iks-overview-done.png)

Or via the CLI,

```bash
$ ibmcloud ks cluster get --cluster $MY_CLUSTER_ID
Retrieving cluster c031jqqd0rll2ksfg97g...
OK
                                   
Name:                           bnewell2-iks118-vpc-cluster1   
ID:                             c031jqqd0rll2ksfg97g   
State:                          normal   
Status:                         All Workers Normal   
Created:                        2021-01-18 23:29:47 +0000 (16 hours ago)   
Resource Group ID:              68af6383f717459686457a6434c4d19f   
Resource Group Name:            Default   
Pod Subnet:                     172.17.0.0/18   
Service Subnet:                 172.21.0.0/16   
Workers:                        1   
Worker Zones:                   us-south-1   
Ingress Subdomain:              bnewell2-iks118-vpc-clu-47d7983a425e05fef831e694b7945b16-0000.us-south.containers.appdomain.cloud   
Ingress Secret:                 bnewell2-iks118-vpc-clu-47d7983a425e05fef831e694b7945b16-0000   
Ingress Status:                 warning   
Ingress Message:                One or more ALBs are unhealthy. See http://ibm.biz/ingress-alb-check   
Creator:                        -   
Public Service Endpoint URL:    https://c111.us-south.containers.cloud.ibm.com:31097   
Private Service Endpoint URL:   https://c111.private.us-south.containers.cloud.ibm.com:31097   
Pull Secrets:                   enabled in the default namespace   
VPCs:                           r006-3c9ab19c-e8af-4eb3-ab60-b9777c3cce1c   

Master         
Status:     Ready (15 hours ago)   
State:      deployed   
Health:     normal   
Version:    1.18.14_1537   
Location:   Dallas   
URL:        https://c111.us-south.containers.cloud.ibm.com:31097 
```

You can also check the status of the Application Load Balancer (ALB),

```console
$ ibmcloud ks ingress alb ls --cluster $MY_CLUSTER_ID

OK
ALB ID                                Enabled   State      Type      Load Balancer Hostname                 Zone         Build                                  Status   
private-crc031jqqd0rll2ksfg97g-alb1   false     disabled   private   -                                      us-south-1   ingress:/ingress-auth:                 -   
public-crc031jqqd0rll2ksfg97g-alb1    true      enabled    public    f128a85f-us-south.lb.appdomain.cloud   us-south-1   ingress:0.35.0_869_iks/ingress-auth:   -   
```

## Deploy the Guestbook Application

Now the cluster is fully provisioned successfully, you can connect to your cluster and set the `current-context` of the `kubectl`,

```bash
ibmcloud ks cluster config --cluster $MY_CLUSTER_ID
kubectl config current-context
```

If you have multiple configuration and contexts, you can easily switch between contexts,

```bash
kubectl config use-context $MY_CLUSTER_NAME/$MY_CLUSTER_ID
```

Deploy the guestbook application,

```console
kubectl create namespace $MY_NAMESPACE
kubectl create deployment guestbook --image=ibmcom/guestbook:v1 -n $MY_NAMESPACE
kubectl expose deployment guestbook --type="LoadBalancer" --port=3000 --target-port=3000 -n $MY_NAMESPACE
```

List the created service for guestbook,

```bash
$ kubectl get svc -n $MY_NAMESPACE

NAME        TYPE           CLUSTER-IP     EXTERNAL-IP                            PORT(S)          AGE
guestbook   LoadBalancer   172.21.48.26   7a6a66a7-us-south.lb.appdomain.cloud   3000:32219/TCP   31m
```

Create environment variables for the public IP address of the `LoadBalancer` service, the `NodePort`, and the port,

```bash
SVC_EXTERNAL_IP=$(kubectl get svc -n $MY_NAMESPACE --output json | jq -r '.items[] | .status.loadBalancer.ingress[0].hostname ')
echo $SVC_EXTERNAL_IP

SVC_NODEPORT=$(kubectl get svc -n $MY_NAMESPACE --output json | jq -r '.items[].spec.ports[] | .nodePort')
echo $SVC_NODEPORT

SVC_PORT=$(kubectl get svc -n $MY_NAMESPACE --output json | jq -r '.items[].spec.ports[] | .port')
echo $SVC_PORT
```

**Note** the external IP address is set to the `hostname` of one of two `Load Balancers for VPC`. See the [Load balancers for VPC](https://cloud.ibm.com/vpc-ext/network/loadBalancers) in the IBM Cloud VPC Infrastructure listing.

![Load balancers fort VPC](images/ibmcloud-loadbalancers-for-vpc.png)

Try to send a request to the guestbook application,

```bash
curl http://$SVC_EXTERNAL_IP:$SVC_PORT

curl: (52) Empty reply from server
```

Even if you have created a `LoadBalancer` service with an external IP, the service cannot be reached because the VPC does not include a rule to allow incoming or ingress traffic to the service.

## Update the Security Group

To allow any traffic to applications that are deployed on your cluster's worker nodes, you have to modify the VPC's default security group by ID.

Update the security group and add an inbound rule for the NodePort of the service you created when exposing the guestbook deployment. I only want to allow ingress traffic on the NodePort, so I set the minimum and maximum value of allowed inbound ports to the same NodePort value.

```bash
$ ibmcloud is security-group-rule-add $MY_DEFAULT_SG_ID inbound tcp --port-min $SVC_NODEPORT --port-max $SVC_NODEPORT

Creating rule for security group r006-b4f498ea-1e71-489e-95f0-0e64cf8d520f under account Remko de Knikker as user b.newell2@remkoh.dev...
                          
ID                     r006-26b196de-5e3a-4a7d-b5fb-9baf9173feb7   
Direction              inbound   
IP version             ipv4   
Protocol               tcp   
Min destination port   32219   
Max destination port   32219   
Remote                 0.0.0.0/0
```

List all the security group rules,

```console
$ ibmcloud is security-group-rules $MY_DEFAULT_SG_ID

Listing rules of security group r006-b4f498ea-1e71-489e-95f0-0e64cf8d520f under account Remko de Knikker as user b.newell2@remkoh.dev...
ID                                          Direction   IP version   Protocol                        Remote   
r006-6c281868-cfbd-46d1-b714-81e085ae2b85   outbound    ipv4         all                             0.0.0.0/0   
r006-b07bbf77-84a0-4e35-90b7-5422f035cf1e   inbound     ipv4         all                             compacted-imprison-clinic-support   
r006-26b196de-5e3a-4a7d-b5fb-9baf9173feb7   inbound     ipv4         tcp Ports:Min=32219,Max=32219   0.0.0.0/0
```

Or add a security group rule to allow inbound TCP traffic on all Kubernetes ports in the range of 30000–32767.

Try again to reach the guestbook application,

```console
curl http://$SVC_EXTERNAL_IP:$SVC_PORT
```

You should now be able to see the HTML response object from the Guestbook application. Open the guestbook URL in a browser to review the web page.

```console
echo http://$SVC_EXTERNAL_IP:$SVC_PORT
```

![Guestbook v1](images/guestbook-v1.png)

If you want to understand better how the load balancing for VPC works, review the optional extra section [Understanding the Load Balancer for VPC](vpcgen2-loadbalancer.md)

You can try removing the inbound rule again to check if the VPC rejects the request again,

```bash
ibmcloud is security-group-rule-delete $MY_DEFAULT_SG_ID $MY_DEFAULT_SG_RULE_ID
```

## Conclusion

You are awesome! You secured your Kubernetes cluster with a Virtual Private Cloud (VPC) and started to air-gap the cluster, blocking direct access to your cluster. Security is an important integral part of any software application development and `airgapping` your cluster by adding a VPC Generation 2 is a first step in securing your cluster, network and containers.

## Cleanup

To conclude, you can choose to delete your Kubernetes resources for this tutorial,

```bash
$ ibmcloud ks cluster rm --cluster $MY_CLUSTER_NAME

Do you want to delete the persistent storage for this cluster? If yes, the data cannot be recovered. If no, you can delete the persistent storage later in your IBM Cloud infrastructure account. [y/N]> y
After you run this command, the cluster cannot be restored. Remove the cluster remkohdev-iks118-vpc-cluster1? [y/N]> y
Removing cluster remkohdev-iks118-vpc-cluster1, persistent storage...
OK
```

Now the Kubernetes cluster is deleted, we need to first remove the public gateway and load balancers from the subnet, then remove the subnet from the VPC, and delete the VPC.

Delete the Gateways, Load Balancers, Network Interfaces, subnet, public gateways, and finally delete the VPC,

```bash
ibmcloud is security-group-rules $MY_DEFAULT_SG_ID --output json 

MY_DEFAULT_SG_RULE_ID=$(ibmcloud is security-group-rules $MY_DEFAULT_SG_ID --output json | jq -r --arg SVC_NODEPORT $SVC_NODEPORT  '.[] | select( .port_max==($SVC_NODEPORT|tonumber)) | .id ')
echo $MY_DEFAULT_SG_RULE_ID
```

Delete the rule for the security group,

```bash
$ ibmcloud is security-group-rule-delete $MY_DEFAULT_SG_ID $MY_DEFAULT_SG_RULE_ID
This will delete security group rule r006-4f5f795f-fa90-44e4-b9c4-cb9457a2a421 and cannot be undone. Continue [y/N] ?> y
Deleting rule r006-4f5f795f-fa90-44e4-b9c4-cb9457a2a421 from security group  r006-db96d593-c224-497d-888c-03d84f6d8e98 under account Remko de Knikker as user b.newell2@remkoh.dev...
OK
Rule r006-4f5f795f-fa90-44e4-b9c4-cb9457a2a421 is deleted.
```

Detach the public gateway from the subnet,

```bash
$ ibmcloud is subnet-public-gateway-detach $MY_VPC_SUBNET_ID
Detaching public gateway from subnet 0717-57ebaf2d-0de6-4630-af01-6cd84031b679 under account Remko de Knikker as user b.newell2@remkoh.dev...
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
Deleting public gateway r006-f4603b78-839b-42a5-949c-76403948821a under account Remko de Knikker as user b.newell2@remkoh.dev...
OK
Public gateway r006-f4603b78-839b-42a5-949c-76403948821a is deleted.
```

Delete the subnet,

```bash
$ ibmcloud is subnet-delete $MY_VPC_SUBNET_ID
This will delete Subnet 0717-57ebaf2d-0de6-4630-af01-6cd84031b679 and cannot be undone. Continue [y/N] ?> y
Deleting subnet 0717-57ebaf2d-0de6-4630-af01-6cd84031b679 under account Remko de Knikker as user b.newell2@remkoh.dev...
OK
Subnet 0717-57ebaf2d-0de6-4630-af01-6cd84031b679 is deleted.
```

Delete the Virtual Private Cloud,

```bash
MY_VPC_ID=$(ibmcloud is vpcs --output json | jq -r '.[] | select( .name=='\"$MY_VPC_NAME\"') | .id ')
echo $MY_VPC_ID

$ ibmcloud is vpc-delete $MY_VPC_IDThis will delete vpc r006-3c9ab19c-e8af-4eb3-ab60-b9777c3cce1c and cannot be undone. Continue [y/N] ?> y
Deleting vpc r006-3c9ab19c-e8af-4eb3-ab60-b9777c3cce1c under account Remko de Knikker as user b.newell2@remkoh.dev...
OK
vpc r006-3c9ab19c-e8af-4eb3-ab60-b9777c3cce1c is deleted.
```
