# ClusterIP

## Add a Service to `helloworld`

Now we have a basic understanding of the different ServiceTypes on Kubernetes, it is time to expose the Deployment of `helloworld` using a Service object. 

Create the Service object with the default type,

```
MY_NS=my-apps

kubectl create -f helloworld-service.yaml -n $MY_NS

service/helloworld created
```

Describe the Service,

```
kubectl describe svc helloworld -n $MY_NS

Name:              helloworld
Namespace:         my-apps
Labels:            app=helloworld
Annotations:       <none>
Selector:          app=helloworld
Type:              ClusterIP
IP:                172.21.49.18
Port:              <unset>  8080/TCP
TargetPort:        http-server/TCP
Endpoints:         172.30.20.145:8080,172.30.20.146:8080,172.30.20.147:8080
Session Affinity:  None
Events:            <none>
```

You see that Kubernetes by default creates a Service of type `ClusterIP`. The service is now available and discoverable, but only within the cluster, using the `Endpoints` and `port` mapping found via the `selector` and `labels`.

Get the endpoints that were created as part of the Service,

```
kubectl get endpoints -n $MY_NS

NAME    ENDPOINTS    AGE
helloworld    172.30.20.145:8080,172.30.20.146:8080,172.30.20.147:8080    42m
```

You can define a Service without a Pod selector to abstract other kinds of backends than Pods. When a Service has no selector, the corresponding Endpoints object is not created automatically. You can manually map the Service to a network address and port by adding an Endpoints object manually, e.g. get and save the `endpoint-helloworld.yaml` to review the definition,

```
kubectl get ep helloworld -n $MY_NS -o yaml > endpoint-helloworld.yaml
```

or if `vi` is the default editor on your client,

```
kubectl edit endpoints helloworld

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

Go to [NodePort](nodeport.md) to learn more about ServiceType NodePort.
