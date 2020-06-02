# Lab01 - Services and ClusterIP

## Create a Service

If you did not already clone the `helloworld` repository to your client, do so now. 

```
$ git clone https://github.com/remkohdev/helloworld.git
$ ls -al
```

Deploy the `helloworld` application,

```
$ cd helloworld
$ kubectl create -f helloworld-deployment.yaml
deployment.apps/helloworld created
```

The `helloworld` application was deployed, that is a Kubernetes `Deployment` object was created. 

```
$ kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/helloworld-5f8b6b587b-kcmnw   1/1     Running   0          69s
pod/helloworld-5f8b6b587b-lwvcs   1/1     Running   0          69s
pod/helloworld-5f8b6b587b-zl6tm   1/1     Running   0          69s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   172.21.0.1   <none>        443/TCP   4h18m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helloworld   3/3     3            3           69s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/helloworld-5f8b6b587b   3         3         3       69s
```

The deployment created also a ReplicaSet with 3 replicas of the pods. Because we did not create a Service for the `helloworld` containers running in pods, they cannot yet be accessed. 

When a Pod is deployed to a worker node, it is assigned a `private IP address` in the 172.30.0.0/16 range. Worker nodes and pods can securely communicate on the private network by using private IP addresses. However, Kubernetes creates and destroys Pods dynamically, which means that the location of the Pods changes dynamically. When a Pod is destroyed or a worker node needs to be re-created, a new private IP address is assigned.

With a `Service` object, you can use built-in Kubernetes `service discovery` to expose Pods. A Service defines a set of Pods and a policy to access those Pods. Kubernetes assigns a single DNS name for a set of Pods and can load balance across Pods. When you create a Service, a set of pods and `EndPoints` are created to manage access to the pods.

The Endpoints object in Kubernetes is the list of IP and port addresses and are created automatically when a Service is created and configured with the pods matching the selector of the Service. A Service can be configured without a selector, in that case Kubernetes does not create an associated Endpoints object.

Let's look at the declaration for the `helloworld-service.yaml`,

```
$ cat helloworld-service.yaml

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

The `spec` defines a few important attributes: `selector` and `ports`. The set of Pods that a Service targets, is determined by the selector and labels. When a Service has no selector, the corresponding `Endpoints` object is not created automatically. This can be useful in cases where you want to define an Endpoint manually, for instance in the case of an external database instance.

The Service maps the incoming `port` to a `targetPort`. By default the `targetPort` is set to the same value as the incoming `port` field. A port definition in Pods can have a name, and you can reference these names in the `targetPort` attribute of a Service instead of the port number. In the Service example of `helloworld`, the `helloworld-deployment.yaml` file should have the corresponding port defined that the Service references by name,

```
$ cat helloworld-deployment.yaml

ports:
- name: http-server
  containerPort: 8080
```

This even works for pods available via the same network protocol on different ports. Kubernetes supports multiple port definitions on a Service object. 

The default protocol for Services is TCP, see other [supported protocols](https://kubernetes.io/docs/concepts/services-networking/service/#protocol-support). 

## ServiceTypes

Before you create a Service for the `helloworld` application, let's first understand what types of services exist. Kubernetes `ServiceTypes` allow you to specify what kind of Service you want. 

The default type is `ClusterIP`. To expose a Service onto an external IP address, you have to create a ServiceType other than ClusterIP.

ServiceType values and their behaviors are:

- **ClusterIP**: Exposes the Service on a cluster-internal IP. This is the default ServiceType.
- **NodePort**: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.
- **LoadBalancer**: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
- **ExternalName**: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record.

You can also use `Ingress` in place of `Service` to expose HTTP/HTTPS Services. Ingress however is technically not a ServiceType, but it acts as the entry point for your cluster and lets you consolidate routing rules into a single resource.

## Add a Service to `helloworld`

Now you have a good understanding of the different ServiceTypes on Kubernetes, it is time to expose the Deployment of `helloworld`, using a Service. 

Create the Service object with the default type,

```
$ kubectl create -f helloworld-service.yaml
service/helloworld created
```

Describe the Service,

```
$ kubectl describe svc helloworld
Name:              helloworld
Namespace:         default
Labels:            app=helloworld
Annotations:       <none>
Selector:          app=helloworld
Type:              ClusterIP
IP:                172.21.161.255
Port:              <unset>  8080/TCP
TargetPort:        http-server/TCP
Endpoints:         172.30.153.79:8080,172.30.50.134:8080,172.30.50.135:8080
Session Affinity:  None
Events:            <none>
```

You see that Kubernetes by default creates a Service of type `ClusterIP`. The service is now available and discoverable, but only within the cluster.

Get the endpoints that were created as part of the Service,

```
$ kubectl get endpoints
NAME         ENDPOINTS                                                  AGE
helloworld   172.30.153.79:8080,172.30.50.134:8080,172.30.50.135:8080   43s
kubernetes   172.20.0.1:2040                                            4h29m
```

You can define a Service without a Pod selector to abstract other kinds of backends than Pods. When a Service has no selector, the corresponding Endpoints object is not created automatically. You can manually map the Service to a network address and port by adding an Endpoints object manually, e.g. 

```
$ kubectl edit endpoints helloworld

apiVersion: v1
kind: Endpoints
metadata:
  name: helloworld
  namespace: default
  labels:
    app: helloworld
subsets:
  - addresses:
      - ip: 172.30.153.79
        targetRef:
          kind: Pod
          name: helloworld-5f8b6b587b-lwvcs 
```

The Endpoints object maps the Service object to a Pod on an internal IP address.

An [ExternalName Service](https://kubernetes.io/docs/concepts/services-networking/service/#externalname) is a special case of Service that does not have selectors and uses DNS names instead, e.g. 

```
apiVersion: v1
kind: Service
metadata:
  name: my-database-svc
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

When looking up the service `my-database-svc.prod.svc.cluster.local`, the cluster DNS Service returns a CNAME record for `my.database.example.com`.

Go to [Lab02](../Lab02/README.md) to learn more about the ServiceType NodePort.

## Kubernetes Networking Extra

To learn more in-depth background information about Kubernetes Networking, go [here](README2.md). 
