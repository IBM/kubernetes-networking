# Setup

## Install Calicoctl

Calicoctl is **not needed** for the current lab.

Install calicoctl,

```console
curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/v3.17.1/calicoctl
chmod +x calicoctl
echo "export PATH=$(pwd):$PATH" > $HOME/.bash_profile
source $HOME/.bash_profile
ibmcloud ks cluster config --cluster $KS_CLUSTER_NAME --admin --network
```

Test the version,

```console
$ calicoctl version

Client Version:    v3.17.1
Git commit:        8871aca3
Cluster Version:   v3.16.5
Cluster Type:      typha,kdd,k8s,operator,openshift,bgp
```

As kubectl plugin,

```console
curl -o kubectl-calico -L  https://github.com/projectcalico/calicoctl/releases/download/v3.17.1/calicoctl
chmod +x kubectl-calico
kubectl calico -h
```
