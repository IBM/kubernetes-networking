# Services

## Create a Service

Clone the `helloworld` repository to your client, 

```console
git clone https://github.com/remkohdev/helloworld.git
ls -al
cd helloworld
```

Create a separate namespace for the deployment,

```console
MY_NS=my-apps
kubectl create namespace $MY_NS
```

Deploy the `helloworld` application,

```console
$ kubectl create -f helloworld-deployment.yaml -n $MY_NS

deployment.apps/helloworld created
```

The `helloworld` application was deployed and a Kubernetes `Deployment` object was created. 

```console
$ kubectl get all -n $MY_NS

NAME    READY    STATUS    RESTARTS    AGE
pod/helloworld-6c76f57b9d-fsbfh    1/1    Running    0    10s
pod/helloworld-6c76f57b9d-gwk5j    1/1    Running    0    10s
pod/helloworld-6c76f57b9d-jfbrz    1/1    Running    0    10s

NAME    READY    UP-TO-DATE    AVAILABLE    AGE
deployment.apps/helloworld    3/3    3    3    10s

NAME    DESIRED    CURRENT    READY    AGE
replicaset.apps/helloworld-6c76f57b9d    3    3    3    10s
```

The deployment consists of a `Deployment` object, a `ReplicaSet` with 3 replicas of `Pods`. 

Because we did not create a Service for the `helloworld` containers running in pods, they cannot yet be readily accessed. 

When a Pod is deployed to a worker node, it is assigned a `private IP address` in the 172.30.0.0/16 range. Worker nodes and pods can securely communicate on the private network by using private IP addresses. However, Kubernetes creates and destroys Pods dynamically, which means that the location of the Pods changes dynamically. When a Pod is destroyed or a worker node needs to be re-created, a new private IP address is assigned.

With a `Service` object, you can use built-in Kubernetes `service discovery` to expose Pods. A Service defines a set of Pods and a policy to access those Pods. Kubernetes assigns a single DNS name for a set of Pods and can load balance across Pods. When you create a Service, a set of pods and `EndPoints` are created to manage access to the pods.

The Endpoints object in Kubernetes is the list of IP and port addresses and are created automatically when a Service is created and configured with the pods matching the `selector` of the Service. A Service can be configured without a selector, in that case Kubernetes does not create an associated Endpoints object.

Let's look at the declaration for the `helloworld-service.yaml`,

```
cat helloworld-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: helloworld
  labels:
    app: helloworld
spec:
  ports:
  - port: 8080
    targetPort: http-server
  selector:
    app: helloworld
```

The `spec` defines a few important attributes for service discovery: `labels`, `selector` and `port`. The set of Pods that a Service targets, is determined by the selector and labels. When a Service has no selector, the corresponding `Endpoints` object is not created automatically. This can be useful in cases where you want to define an Endpoint manually, for instance in the case of an external database instance.

The Service maps the incoming `port` to athe container's `targetPort`. By default the `targetPort` is set to the same value as the incoming `port` field. A port definition in Pods can also have a name, and you can reference these names in the `targetPort` attribute of a Service instead of the port number. In the Service example of `helloworld`, the `helloworld-deployment.yaml` file should have the corresponding port defined that the Service references by name,

```
cat helloworld-deployment.yaml

ports:
- name: http-server
  containerPort: 8080
```

This even works for pods available via the same network protocol on different port numbers, as Kubernetes supports multiple port definitions on a Service object. 

The default protocol for Services is TCP, see other [supported protocols](https://kubernetes.io/docs/concepts/services-networking/service/#protocol-support). 

## ServiceTypes

Before you create a Service object for the `helloworld` application, let's first review the types of services. Kubernetes `ServiceTypes` allow you to specify what kind of Service you want. 

The default type is `ClusterIP`. To expose a Service onto an external IP address, you have to create a ServiceType other than ClusterIP.

Available Service types:

- **ClusterIP**: Exposes the Service on a cluster-internal IP. This is the default ServiceType.
- **NodePort**: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.
- **LoadBalancer**: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
- **ExternalName**: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record.

You can also use `Ingress` in place of `Service` to expose HTTP/HTTPS Services. Ingress however is technically not a ServiceType, but it acts as the entry point for your cluster and lets you consolidate routing rules into a single resource.

Next, go to [ClusterIP](clusterip.md) to learn more about ServiceType ClusterIP.