# Kubernetes Networking Lab

In this tutorial we'll demonstrate basic networking concepts in Kubernetes using the guestbook application.


## Objectives

In this lab you'll learn
* how pods communicate with each other;
* how pods are exposed to the internet;
* how traffic between pods can be restricted.


## Prerequisites

Before you begin, you need to install the required CLIs to manage your Kubernetes clusters.
IBM provides an installer [here](https://clis.ng.bluemix.net/ui/home.html) to get all of these tools together.
There are instructions for how to obtain the tools manually if desired.  The following tools are used in this tutorial.
* kubectl CLI
   * `kubectl` is a command line interface for running commands against Kubernetes clusters.
* ibmcloud CLI
   * `ibmcloud` is a command line interface for managing resources in IBM Cloud.

This tutorial depends upon features such as an ingress controller which require a standard Kubernetes cluster.
In order to create a standard cluster you must have either a Pay-As-You-Go or a Subscription IBM Cloud account.
See https://console.bluemix.net/docs/account/index.html#accounts for further information about account types.


## Download the Sample Application
The application used in this tutorial is a simple guestbook website where users can post messages.
You should clone it to your workstation since you will be using some of the configuration files.

```console
$ git clone https://github.com/IBM/guestbook.git
```


## Create a standard cluster using IBM Cloud Kubernetes Service

It is recommended to use the [IBM Cloud dashboard](https://console.bluemix.net/catalog/) to create a standard cluster because it will help guide you through
the configuration options and it will show you the estimated cost.
* Click the login button in the upper right and follow the login prompts to log into your account.
* Click the `Containers` tab on the left side of the window and then select the "Kubernetes Service" tile.
* Click the `Create` button.
* Fill in the form that appears.  First select a location for your cluster as this will dictate the other options.
Then choose a zone within the location and a machine type.  Set the number of worker nodes to 2.
Give a name to your cluster; for this tutorial we'll use "myStandardCluster".
* Review the cost estimate on the right side of the window.
* Click the `Create Cluster` button.

Now we'll continue using commands.
Log in to the IBM Cloud CLI and enter your IBM Cloud credentials when prompted.

```console
ibmcloud login
```

Note: If you have a federated ID, use `ibmcloud login --sso` to log in to the IBM Cloud CLI.

Check the status of your cluster as follows.
```console
$ ibmcloud ks clusters
OK
Name                ID                                 State     Created          Workers   Location    Version
myStandardCluster   fc5514ef25ac44da9924ff2309020bb3   normal    12 minutes ago   2         Dallas     1.10.7_1520
```

If the cluster state is `pending`, wait for a moment and try the command again.
Once the cluster is provisioned (state is `normal`), the kubernetes client CLI `kubectl` needs to be configured to talk to the provisioned cluster.
Run `ibmcloud ks cluster-config myStandardCluster` which will create a config file on your workstation.

```console
$ ibmcloud ks cluster-config myStandardCluster
OK
The configuration for mycluster was downloaded successfully. Export environment
variables to start using Kubernetes.

export KUBECONFIG=/home/gregd/.bluemix/plugins/container-service/clusters/myStandardCluster/kube-config-hou02-myStandardCluster.yml
```

Copy the `export` statement that you get and run it.  This sets the `KUBECONFIG` environment variable to point to the kubectl config file.
This will make your `kubectl` client work with your new Kubernetes cluster.  You can verify that by entering a `kubectl` command.

```console
$ kubectl get nodes
NAME            STATUS    ROLES     AGE       VERSION
10.177.184.185   Ready     <none>    2m        v1.10.7+IKS
10.177.184.220   Ready     <none>    2m        v1.10.7+IKS
```


## Deploy the guestbook application

The following figure depicts the components of the sample guestbook application.

![w_to_master-r_to_slave](images/Master-Slave.png)

The guestbook application is a Go program which runs inside a container that is deployed to the Kubernetes cluster.
It includes a simple HTML page and javascript file for a browser front-end.
It uses Redis to store its data. It writes data to a Redis master instance and reads data from multiple Redis slave instances.
The master and slave instances also run as containers that are deployed to the Kubernetes cluster.

Let's first use the configuration files in the cloned Git repository to deploy the containers and create services for them.
We'll then explore the resulting environment.

```console
$ cd guestbook/v1

$ kubectl create -f redis-master-deployment.yaml
deployment.apps/redis-master created

$ kubectl create -f redis-master-service.yaml
service/redis-master created

$ kubectl create -f redis-slave-deployment.yaml
deployment.apps/redis-slave created

$ kubectl create -f redis-slave-service.yaml
service/redis-slave created

$ kubectl create -f guestbook-deployment.yaml
deployment.apps/guestbook-v1 created

$ kubectl create -f guestbook-service.yaml
service/guestbook created
```


## Pod network

Let's look at the pods that were created.
The guestbook deployment yaml file requested 3 replicas.
The redis-master deployment yaml file requested a single replica.
The redis-slave deployment yaml file requested 2 replicas.

```console
$ kubectl get pods -o wide
NAME                            READY     STATUS    RESTARTS   AGE      IP               NODE
guestbook-v1-7fc76dc46-bl7xf    1/1       Running   0          2m       172.30.108.141   10.177.184.220
guestbook-v1-7fc76dc46-ccfgs    1/1       Running   0          2m       172.30.58.207    10.177.184.185
guestbook-v1-7fc76dc46-g7hq2    1/1       Running   0          2m       172.30.108.142   10.177.184.220
redis-master-5d8b66464f-p76z8   1/1       Running   0          4m       172.30.108.139   10.177.184.220
redis-slave-586b4c847c-ll9gc    1/1       Running   0          4m       172.30.108.140   10.177.184.220
redis-slave-586b4c847c-twjdb    1/1       Running   0          4m       172.30.58.206    10.177.184.185
```

There are two networks.

* The nodes exist on a private VLAN which is part of your infrastructure account.
  (They also exist on a public VLAN which we'll discuss later.)

* The pods exist on a virtual network which is created by Kubernetes
  (technically by a container networking plugin to Kubernetes called Calico).
  Each pod is assigned a unique IP and can talk to any other pod using its IP.

You can observe how the pod network operates over the host node network by running a traceroute command in one of the pods.
(Note: The ability to run Linux commands within a container varies depending on the base image it uses.
This command will work with guestbook but may not work with other containers.)

```console
$ kubectl exec -it guestbook-v1-7fc76dc46-bl7xf traceroute 172.30.58.206
traceroute to 172.30.58.206 (172.30.58.206), 30 hops max, 46 byte packets
 1  10.177.184.220 (10.177.184.220)  0.007 ms  0.005 ms  0.004 ms
 2  10.177.184.185 (10.177.184.185)  0.652 ms  0.360 ms  0.510 ms
 3  172.30.58.206 (172.30.58.206)  0.591 ms  0.347 ms  0.499 ms
```

Here we're running a traceroute command on pod `guestbook-v1-7fc76dc46-bl7xf` which is running on node `10.177.184.220`.
We ask it to trace the route to another pod, `redis-slave-586b4c847c-twjdb` which has a pod IP address of `172.30.58.206`.
We can see the path goes from the first's pod node over to the second pod's node and into the second pod.
This is accomplished through the use of virtual ethernet interfaces and updates to Linux routing tables.

One benefit of assigning each pod its own IP address is that applications can use standard ports (80, 443, etc)
without the need to remap them at the node level to avoid port conflicts.


## ClusterIP Services

Although pods can communicate with each other using their IP addresses, we wouldn't want application programs to use them
directly.  A pod's IP address can change each time the pod is recreated.  If the pod is scaled, then the set of IP addresses to
communicate with can be changing frequently.

This is where the ClusterIP services comes in.  Let's look at the services we created for the redis master and slaves.

```console
$ kubectl describe service redis-master
Name:              redis-master
Namespace:         default
Labels:            app=redis
                   role=master
Annotations:       <none>
Selector:          app=redis,role=master
Type:              ClusterIP
IP:                172.21.193.142
Port:              <unset>  6379/TCP
TargetPort:        redis-server/TCP
Endpoints:         172.30.108.139:6379
Session Affinity:  None
Events:            <none>

$ kubectl describe service redis-slave
Name:              redis-slave
Namespace:         default
Labels:            app=redis
                   role=slave
Annotations:       <none>
Selector:          app=redis,role=slave
Type:              ClusterIP
IP:                172.21.60.238
Port:              <unset>  6379/TCP
TargetPort:        redis-server/TCP
Endpoints:         172.30.108.140:6379,172.30.58.206:6379
Session Affinity:  None
Events:            <none>
```

A ClusterIP service provides a stable virtual IP address which distributes TCP connections (or UDP datagrams) to a targeted set of pods (called endpoints).
Here we can see that the redis-master service has the virtual IP address `172.21.193.14` and that it distributes requests to the redis-master's pod IP address `172.30.108.139`.
The redis-slave service has a virtual IP address `172.21.60.238` and it distributes requests to the redis-slave's pod IP addresses `172.30.108.140` and `172.30.58.206`.

The method by which Kubernetes implements the virtual IP address varies by Kubernetes release.  In the 1.10 release used in this tutorial the default method is to
use iptables to translate (NAT) the virtual IP addresses to the pod IP addresses.

## Service discovery

Kubernetes provides a DNS entry for each service so services can be addressed by name instead of IP address.
Let's observe this by doing an `nslookup` command from within the container.
(Note: The ability to run Linux commands within a container varies depending on the base image it uses.
This command will work with guestbook but may not work with other containers.)

```console
$ kubectl exec -it guestbook-v1-7fc76dc46-bl7xf nslookup redis-master
Server:    172.21.0.10
Address 1: 172.21.0.10 kube-dns.kube-system.svc.cluster.local

Name:      redis-master
Address 1: 172.21.193.142 redis-master.default.svc.cluster.local
```

Here we see that the name redis-master is resolved to address `172.21.193.142` which is the virtual IP address of the
redis-master service.  There is a long-form name redis-master.default.svc.cluster.local as well.  The long-form name
is needed to address services across namespaces.  In this tutorial we are only using the `default` namespace so the
long form isn't needed.


## NodePort and LoadBalancer Services

The types of IPs presented so far, pod IPs and ClusterIPs, are usable only from within the Kubernetes cluster.
It is not possible for applications outside the cluster to use them to reach a pod.
For that we need to use a type of service which provides an external IP address.
Kubernetes provides two service types which do this.

* A `NodePort` service exposes the service at a static port (the NodePort) on each node.  A ClusterIP service, to which the NodePort service will route, is automatically created.

* A `LoadBalancer` service exposes the service externally using a cloud provider's load balancer.  NodePort and ClusterIP services, to which the external load balancer will route, are automatically created.

Let's take a look at the service which we created for the guestbook application.

```console
$ kubectl describe service guestbook
Name:                     guestbook
Namespace:                default
Labels:                   app=guestbook
Annotations:              <none>
Selector:                 app=guestbook
Type:                     LoadBalancer
IP:                       172.21.189.71
LoadBalancer Ingress:     169.46.35.163
Port:                     <unset>  3000/TCP
TargetPort:               http-server/TCP
NodePort:                 <unset>  30347/TCP
Endpoints:                172.30.108.141:3000,172.30.108.142:3000,172.30.58.207:3000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
  Normal  EnsuringLoadBalancer  20s   service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   20s   service-controller  Ensured load balancer
```

This is a LoadBalancer type of service.  The services build upon each other so a LoadBalancer service is also a
NodePort service and a ClusterIP service.  Let's break this down.

* A ClusterIP (identified by the `IP` label) was assigned and it is `172.21.189.71`.
* A NodePort was assigned and it is 30347.
* A LoadBalancer IP address was assigned and it is `169.46.35.163`.

We now have two ways to access the guestbook application from outside the cluster, either through a node's IP address or through the load balancer's IP address.
The latter is straightforward:  use the loadbalancer address and the port that's configured by the service, which in this case is `3000`.

```console
http://169.46.35.163:3000
```

Using the NodePort service requires us to first find the public addresses of the worker nodes.

```console
$ ibmcloud ks workers myStandardCluster
OK
ID                                                 Public IP        Private IP       Machine Type        State    Status   Zone    Version
kube-dal10-crfc5514ef25ac44da9924ff2309020bb3-w1   169.47.252.42    10.177.184.220   u2c.2x4.encrypted   normal   Ready    dal10   1.10.7_1520*
kube-dal10-crfc5514ef25ac44da9924ff2309020bb3-w2   169.48.165.242   10.177.184.185   u2c.2x4.encrypted   normal   Ready    dal10   1.10.7_1520*
```

We need to combine the worker node's IP address with the NodePort assigned by Kubernetes which in this example is `30347`.  The NodePort gives
the service a unique endpoint on the node.

```console
http://169.47.252.42:30347
http://169.48.165.242:30347
```

Note that a NodePort service also distributes TCP connections to the pods.  It does not require a pod to be running on the node which you address.
If there is a pod running on the node which you address, it is not always the case that a connection will be routed to that pod if there are pods
on other nodes as well.

The NodePort service typically would be used only in the following cases:
* in cloud environments which don't support a LoadBalancer service (such as in a trial account on IBM Cloud or in a local environmment such as `minikube`);
* in development environments where you don't need or want to pay the cost of the LoadBalancer service.


## Ingress

Kubernetes LoadBalancer services are primitive.
They do not provide the functional capabilities typically associated with a
public-facing application load balancer such as SSL termination and client authentication.
For these features you need to use an Ingress resource instead of a LoadBalancer service.
Let's change the guestbook application for using a LoadBalancer service to using an Ingress resource.
First let's delete the existing guestbook service.

```console
$ kubectl delete service guestbook
service "guestbook" deleted
```

We'll use the following yaml to define a new service and Ingress resource.

```console
apiVersion: v1
kind: Service
metadata:
  name: guestbook
  labels:
    app: guestbook
spec:
  ports:
  - port: 3000
    targetPort: http-server
  selector:
    app: guestbook
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: guestbook
  annotations:
    ingress.bluemix.net/rewrite-path: "serviceName=guestbook rewrite=/"
spec:
  tls:
  - hosts:
    - <ingress-subdomain>
    secretName: <ingress-secret>
  rules:
  - host: <ingress-subdomain>
    http:
      paths:
      - path: /guestbook/
        backend:
          serviceName: guestbook
          servicePort: 3000
```

You can find this yaml [here](guestbook-ingress.yaml).  Download it and open it in a text editor.

There are a couple of updates that need to be made to the yaml before we can give it to Kubernetes.
Run the following command:

```console
$ ibmcloud ks cluster-get myStandardCluster
Retrieving cluster myStandardCluster...
OK


Name:                   myStandardCluster
ID:                     fc5514ef25ac44da9924ff2309020bb3
State:                  normal
Created:                2018-09-18T13:58:23+0000
Location:               dal10
Master URL:             https://169.60.128.2:24627
Master Location:        Dallas
Ingress Subdomain:      mystandardcluster.us-south.containers.appdomain.cloud
Ingress Secret:         mystandardcluster
Workers:                2
Worker Zones:           dal10
Version:                1.10.7_1520
Owner Email:
Monitoring Dashboard:   -
```

There are two values of interest in the output (the values in your output may be different):

* Ingress Subdomain:  IBM provides this domain name which you can use to access your application from the internet.

* Ingress Secret:  IBM provides certificates for the IBM-provided domain name.  The certificates and private key are
  stored in this Kubernetes secret.

There are ways to use your own domain name and certificates but for this tutorial we'll use the ones provided by IBM.

Update your copy of the yaml file to use the values you obtained.
Replace `<ingress-subdomain>` with the Ingress Subdomain and replace `<ingress-secret>` with the Ingress Secret.
Save the file and then tell Kubernetes to create the resources defined by your copy of the yaml file.

```console
$ kubectl create -f <location-of-your-copy-of-yaml-file>
service/guestbook created
ingress.extensions/guestbook created
```

Let's walk through what happens with the Ingress resource.  Behind the scenes your cluster contains an application
load balancer which in the IBM Cloud is provided by `nginx`.  A Kubernetes component called an Ingress controller takes
your Ingress resource and translates it to a corresponding application load balancer configuration.

The Ingress resource needs to tell the load balancer which services to expose and how to identify which request should
be routed to which service.  If you have only one service that will be ever be exposed, this is relatively easy:  all requests
to your ingress subdomain should be routed to that one service.  However it's likely that you'll have more than one service.
Furthermore most web applications are written to serve files starting at the root path (/) so a way is needed to separate them.
The Ingress resource we're using accomplishes this as follows:

* It defines a `path` which tells the load balancer that a request that begins with `/guestbook/` should be routed to the guestbook service.

* It uses an annotation `ingress.bluemix.net/rewrite-path` which tells the load balancer that when sending requests
  to the guestbook application, it needs to rewrite the path to replace `/guestbook/` with `/`.

This pattern can be repeated as you expose more services to the internet.


## Network Policies

Kubernetes network policies specify how pods can communicate with other pods and with external endpoints.
Default network policies are set up to secure your Kubernetes cluster.
If you have unique security requirements, you can create your own network policies.

The following network traffic is allowed by default:
* A pod accepts external traffic from any IP address to its NodePort or LoadBalancer service or its Ingress resource.
* A pod accepts internal traffic from any other pod in the same cluster.
* A pod is allowed outbound traffic to any IP address.

Network policies let you create additional restrictions on what traffic is allowed.
For example you may want to restrict external inbound or outbound traffic to certain IP addresses.

For this tutorial we'll use a network policy to restrict traffic between pods.
Let's say that we want to limit access to the redis servers to the guestbook application.
First we can observe that the redis servers are open to any pod by spinning up a Linux shell
inside a pod and making a network connection to the redis servers' IP addresses and ports.

```console
$ kubectl get pods -o wide
NAME                            READY     STATUS    RESTARTS   AGE       IP               NODE
guestbook-v1-7fc76dc46-bl7xf    1/1       Running   0          4d        172.30.108.141   10.177.184.220
guestbook-v1-7fc76dc46-ccfgs    1/1       Running   0          4d        172.30.58.207    10.177.184.185
guestbook-v1-7fc76dc46-g7hq2    1/1       Running   0          4d        172.30.108.142   10.177.184.220
redis-master-5d8b66464f-p76z8   1/1       Running   0          4d        172.30.108.139   10.177.184.220
redis-slave-586b4c847c-ll9gc    1/1       Running   0          4d        172.30.108.140   10.177.184.220
redis-slave-586b4c847c-twjdb    1/1       Running   0          4d        172.30.58.206    10.177.184.185

$ kubectl run -i --tty --rm busybox --image=busybox -- sh
If you don't see a command prompt, try pressing enter.
/ # nc -v -z 172.30.108.139 6379
172.30.108.139 (172.30.108.139:6379) open
/ # nc -v -z 172.30.108.140 6379
172.30.108.140 (172.30.108.140:6379) open
/ # nc -v -z 172.30.58.206 6379
172.30.58.206 (172.30.58.206:6379) open
/ # exit
Session ended, resume using 'kubectl attach busybox-5858cc4697-hb6zs -c busybox -i -t' command when the pod is running
deployment.apps "busybox" deleted
```

First we used the `kubectl get pods -o wide` command to see the IP addresses assigned to the redis pods.
Then we ran a busybox image to try `nc` commands to each of those addresses.
(The redis server ports are defined in the redis-master-service.yaml and redis-slave-service.yaml files we used earlier.
The `nc` commands show that connections are allowed.

Now let's define a network policy to restrict access to the redis servers to the guestbook application.

```console
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: redis-policy
spec:
  podSelector:
    matchLabels:
      app: redis
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: guestbook
```

You can find this yaml [here](guestbook-network-policy.yaml).  Download it to your workstation and tell Kubernetes to create it.



```console
$ kubectl create -f <location-of-your-copy-of-yaml-file>
networkpolicy.networking.k8s.io/redis-policy created
```

Now we can repeat the `nc` commands and see that connections to the redis servers are not allowed from an arbitrary pod.
The guestbook application continues to work though.

```console
$ kubectl run -i --tty --rm busybox --image=busybox -- sh
If you don't see a command prompt, try pressing enter.
/ # nc -v -z 172.30.108.139 6379
nc: 172.30.108.139 (172.30.108.139:6379): Connection timed out
/ # nc -v -z 172.30.108.140 6379
nc: 172.30.108.140 (172.30.108.140:6379): Connection timed out
/ # nc -v -z 172.30.58.206 6379
nc: 172.30.58.206 (172.30.58.206:6379): Connection timed out
/ # exit
Session ended, resume using 'kubectl attach busybox-5858cc4697-mvgsf -c busybox -i -t' command when the pod is running
deployment.apps "busybox" deleted
```
