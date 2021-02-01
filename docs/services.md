# Services

This tutorial will demonstrate different ways to control traffic on a Kubernetes cluster using Service types, Ingress, Route, Network Policy and Calico. You will learn basic networking and security concepts on Kubernetes.

## Setup

For setup and pre-requisities, go [here](setup1.md).

Make sure you are connected to the Kubernetes cluster.

```
$ oc config current-context

my-apps/c115-e-us-south-containers-cloud-ibm-com:32370/IAM#remkohdev@us.ibm.com
```

If you are not connected to your cluster, finish the [setup](setup1.md).

## Deploy the Helloworld App

To deploy the application to a Kubernetes cluster in an isolated namespace, we create a separate namespace for the application. Throughout the tutorial I will use environment variables in the commands. 

First, create an environment variable for a namespace to deploy our application,

```console
MY_NS=my-apps
echo $MY_NS
```

Create the namespace,

```console
$ oc new-project $MY_NS

Now using project "my-apps" on server "https://c109-e.us-east.containers.cloud.ibm.com:31345".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app ruby~https://github.com/sclorg/ruby-ex.git

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
```

Or if you already created the namespace, switch to the namespace where you want to deploy the application to,

```
$ oc project $MY_NS

Now using project "my-apps" on server "https://c109-e.us-east.containers.cloud.ibm.com:31345".
```

In this workshop, we use a `helloworld` application. The source code and Kubernetes resource specifications are included in the repository. Make sure, you cloned the `helloworld` repository to your client, 

```console
git clone https://github.com/remkohdev/helloworld.git
cd helloworld
ls -al
```

Now you can deploy the `helloworld` application in an isolated namespace, this is the first step to create some level of isolation and limit a possible attack surface,

```console
$ oc create -f helloworld-deployment.yaml -n $MY_NS

deployment.apps/helloworld created
```

On OpenShift you do not have to specify the namespace to deploy to the current namespace, but for clarity I added the flag `-n $MY_NS`.

If you don't have the source code for the Helloworld app with the Kubernetes specification files, go to the [setup](setup1.md) to `Get Helloworld Source Code`.

The `helloworld` application is now deployed and a Kubernetes `Deployment` object was created. 

```console
$ oc get all -n $MY_NS

NAME                              READY   STATUS    RESTARTS   AGE
pod/helloworld-5f8b6b587b-qqqjs   1/1     Running   0          73s
pod/helloworld-5f8b6b587b-rwkf9   1/1     Running   0          73s
pod/helloworld-5f8b6b587b-tbpts   1/1     Running   0          73s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helloworld   3/3     3            3           73s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/helloworld-5f8b6b587b   3         3         3       73s
```

The deployment consists of a `Deployment` object, a `ReplicaSet` with 3 replicas of `Pods`. 

Because we did not create a Service for the `helloworld` containers running in pods, they cannot yet be readily accessed. 

When a Pod is deployed to a worker node, it is assigned a `private IP address` in the 172.30.0.0/16 range. Worker nodes and pods can securely communicate on the private network by using private IP addresses. However, Kubernetes creates and destroys Pods dynamically, which means that the location of the Pods changes dynamically. When a Pod is destroyed or a worker node needs to be re-created, a new private IP address is assigned.

With a `Service` object, you can use built-in Kubernetes `service discovery` to expose Pods. A Service defines a set of Pods and a policy to access those Pods. Kubernetes assigns a single DNS name for a set of Pods and can load balance across Pods. When you create a Service, a set of pods and `EndPoints` are created to manage access to the pods.

The Endpoints object in Kubernetes contains a list of IP and port addresses and are created automatically when a Service is created and configured with the pods matching the `selector` of the Service.

## ServiceTypes

Before we create the Service for the `helloworld` application, briefly review the types of services.

The default type is `ClusterIP`. To expose a Service onto an external IP address, you have to create a ServiceType other than ClusterIP.

Available Service types:

- **ClusterIP**: Exposes the Service on a cluster-internal IP. This is the default ServiceType.
- **NodePort**: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.
- **LoadBalancer**: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
- **ExternalName**: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record.

You can also use `Ingress` in place of `Service` to expose HTTP/HTTPS Services. Ingress however is technically not a ServiceType, but it acts as the entry point for your cluster and lets you consolidate routing rules into a single resource. 

A `Route` is similar to `Ingress` and was created on OpenShift before the introduction of `Ingress` in Kubernetes. Route has additional capabilities such as splitting traffic between multiple backends and sticky sessions. Route design principles heavily influenced the Ingress design.

## Next

Next, go to [ClusterIP](clusterip.md) to learn more about ServiceType ClusterIP.