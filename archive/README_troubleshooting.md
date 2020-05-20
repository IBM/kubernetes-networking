When creating a LoadBalancer service on a 3nodes IKS cluster, I got the following error:
```
Error on cloud load balancer a0a497a3221324f90a6058366df4c9a9 for service default/guestbook 
with UID 0a497a32-2132-4f90-a605-8366df4c9a9b: 
No cloud provider IPs are available to fulfill the load balancer service request. 

Add a portable subnet to the cluster and try again. 
See https://cloud.ibm.com/docs/containers?topic=containers-cs_troubleshoot_network#cs_troubleshoot_network 
for details.
```

To resolve it, add a portable subnet to the cluster as follows:
```
$ CLUSTER_NAME=remkohdev-iks116-3x-cluster
$ ibmcloud ks cluster get --cluster  $CLUSTER_NAME --show-resources --show-resources
Subnet VLANs
VLAN ID   Subnet CIDR         Public   User-managed
2832802   169.47.196.184/29   true     false
2832804   10.171.129.48/29    false    false
$ PUBLIC_SUBNET_CIDR=‘169.47.196.184/29’
$ PUBLIC_VLAN_ID=2832802
$ ibmcloud ks subnets --provider classic | grep $PUBLIC_VLAN_ID
$ ibmcloud ks subnets --provider classic | grep $PUBLIC_SUBNET_CIDR
1367227   169.47.196.184/29    169.47.196.185    2832802   public    bqfo73dd0ajmhsng0e5g   dal10
$ PUBLIC_SUBNET_ID=1367227
$ ibmcloud ks cluster subnet add --cluster $CLUSTER_NAME --subnet-id $PUBLIC_SUBNET_ID
$ ibmcloud ks cluster get --cluster $CLUSTER_NAME  --show-resources
```