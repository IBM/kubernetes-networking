# Services

This tutorial will demonstrate different ways to control traffic on a Kubernetes cluster using Service types, Ingress, Route, Network Policy and Calico. You will learn basic networking and security concepts on Kubernetes.

## Setup

For setup and pre-requisities, go [here](setup1.md).

Make sure you are connected to the Kubernetes cluster and you're in the desired namespace or project.

```bash
$ oc config current-context

my-apps/c115-e-us-south-containers-cloud-ibm-com:32370/IAM#remkohdev@us.ibm.com
```

If you are not connected to your cluster, finish the [setup](setup1.md).

## Users, Namespaces and Projects

To deploy the application to a Kubernetes cluster in an isolated namespace, we create a separate namespace for the application to isolate our resources.

Create an environment variable for the namespace or project to use in this tutorial,

```bash
MY_NS=my-apps
echo $MY_NS
```

If you did not create the namespace yet, create the namespace,

```bash
$ oc new-project $MY_NS

Now using project "my-apps" on server "https://c109-e.us-east.containers.cloud.ibm.com:31345".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app ruby~https://github.com/sclorg/ruby-ex.git

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
```

Or if you already created the namespace, switch to the namespace where you want to deploy the application to,

```bash
$ oc project $MY_NS

Now using project "my-apps" on server "https://c109-e.us-east.containers.cloud.ibm.com:31345".
```

Interaction with OpenShift is based on a `user` who is granted permissions through roles. Users can be a regular `User`, system users (e.g. system:admin, system:openshift-registry) or service accounts, a special system user `ServiceAccount` associated with a `project`. A user can be assigned to groups (user defined groups, system groups or virtual groups).

A Kubernetes `namespace` allows scoping of resources. A `project` is a namespace with additional annotations to isolate their resources. A project scopes its own policies, constraints and service accounts.

```bash
$ oc describe project $MY_NS
Name:                   my-appsCreated:                19 minutes ago
Labels:                 <none>Annotations:            openshift.io/description=
                        openshift.io/display-name=
                        openshift.io/requester=IAM#remkohdev@us.ibm.com
                        openshift.io/sa.scc.mcs=s0:c25,c15
                        openshift.io/sa.scc.supplemental-groups=1000630000/10000
                        openshift.io/sa.scc.uid-range=1000630000/10000
Display Name:           <none>
Description:            <none>
Status:                 Active
Node Selector:          <none>
Quota:                  <none>
Resource limits:        <none>
```

OpenShift allocated 3 ranges to the project: mcs, supplemental-groups, and uid-range.

* [Multi-Category Security (MCS)](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/deployment_guide/sec-mcs-ov) are SELinux labels that represents the owner of the process (defined separately from the users in the system) and is comprised of 3 unique sub-values sX:cY,cZ for each process-owner. MCS labels are used to distinguish between containers in different namespaces.
* GID range, Supplemental-Group for shared storage (NFS/GlusterFS), fsGroup for block storage (Ceph RBD, iSCSI, etc),
* UID, UID range.

A GID is unique to a pod. A UID is unique to a container.

Inside your project,

```bash
$ oc get sa
NAME       SECRETS   AGE
builder    2         27m
default    2         27m
deployer   2         27m
```

For example, the ServiceAccounts `builder`, `default` and `deployer` are created when you create a new namespace (`builder` is responsible for S2I image building, `deployer` is responsible for the number of pods in a replicaset and `default` runs the pod and container on the host). You can also create custom ServiceAccounts for your resources.

The `admission controller` enforces the security policies for each resource, like RBAC and Security Context Constraints (SCC).

## Deploy the Helloworld App

In this workshop, we use a `helloworld` application. The source code and Kubernetes resource specifications are included in the repository. Make sure, you cloned the `helloworld` repository to your client.

```bash
git clone https://github.com/remkohdev/helloworld.git
cd helloworld
ls -al
```

### Deployment

Now you can deploy the `helloworld` application in an isolated namespace, this is the first step to create some level of isolation and limit a possible attack surface,

```bash
$ oc create -f helloworld-deployment.yaml -n $MY_NS

deployment.apps/helloworld created
```

On OpenShift you do not have to specify the namespace to deploy to the current namespace, but for clarity the flag `-n $MY_NS` was added.

The `helloworld` application is now deployed and a Kubernetes `Deployment` object was created.

```bash
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

### Service

Because we did not create a Service for the `helloworld` containers running in pods, they cannot yet be easily accessed. When a Pod is deployed to a worker node, it is assigned a `private IP address` in the 172.30.0.0/16 range. Worker nodes and pods can securely communicate on the private network by using private IP addresses. However, Kubernetes creates and destroys Pods dynamically, which means that the location of the Pods changes dynamically. When a Pod is destroyed or a worker node needs to be re-created, a new private IP address is assigned.

With a `Service` object, you can use built-in Kubernetes `service discovery` to expose Pods. A Service defines a set of Pods and a policy to access those Pods. Kubernetes assigns a single DNS name for a set of Pods and can load balance across Pods. When you create a Service, a set of pods and `EndPoints` are created to manage access to the pods.

The Endpoints object in Kubernetes contains a list of IP and port addresses and are created automatically when a Service is created and configured with the pods matching the `selector` of the Service.

## ServiceTypes

Before we create the Service for the `helloworld` application, briefly review the types of services.

The default type is `ClusterIP`. To expose a Service onto an external IP address, you have to create a ServiceType other than ClusterIP.

Available Service types:

* **ClusterIP**: Exposes the Service on a cluster-internal IP. This is the default ServiceType.
* **NodePort**: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting `NodeIP`:`NodePort`.
* **LoadBalancer**: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
* **ExternalName**: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record.

You can also use `Ingress` in addition to `Service` to expose HTTP/HTTPS Services. Ingress however is technically not a ServiceType. It acts as the entry point for your cluster and lets you consolidate routing rules into a single resource.

A `Route` is similar to `Ingress` and was created for OpenShift before the introduction of `Ingress` in Kubernetes. Route has additional capabilities such as splitting traffic between multiple backends and sticky sessions. Route design principles heavily influenced the Ingress design.

## Next

Next, go to [ClusterIP](clusterip.md) to learn more about ServiceType ClusterIP.
