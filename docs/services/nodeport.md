# NodePort

## Pre-requisites

Finish the [Services](services.md) and [ClusterIP](clusterip.md) labs.

## NodePort

To expose a Service to an external IP address, you have to create a ServiceType other than ClusterIP. When you send a request to the name of the service, `kube-proxy` looks up the name in the cluster `DNS server` and routes the request to the in-cluster IP address of the service. 

To allow external traffic into a kubernetes cluster, you need a `NodePort` ServiceType. When kubernetes creates a NodePort service, `kube-proxy` allocates a port in the range **30000-32767** and opens this port on the `eth0` interface of every node (the `NodePort`). Connections to this port are then forwarded to the service’s cluster IP. A gateway router typically sits in front of the cluster and forwards packets to the node.

Patch the existing Service for `helloworld` to `type: NodePort` using the `kubectl patch` command,

```console
$ oc patch svc helloworld -p '{"spec": {"type": "NodePort"}}' -n $MY_NS

service/helloworld patched
```

You can also add a property `type: NodePort` in the specification in the file `helloworld-service-nodeport.yaml`,

```console
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
  type: NodePort
```

To apply changes to the configuration from file,

```console
$ oc apply -f helloworld-service-nodeport.yaml -n $MY_NS

Warning: oc apply should be used on resource created by either oc create --save-config or oc apply
service/helloworld configured
```

Describe the Service,

```console
$ oc describe svc helloworld -n $MY_NS

Name:                     helloworld
Namespace:                my-apps
Labels:                   app=helloworld
Annotations:              Selector:  app=helloworld
Type:                     NodePort
IP:                       172.21.86.16
Port:                     <unset>  8080/TCP
TargetPort:               http-server/TCP
NodePort:                 <unset>  32387/TCP
Endpoints:                172.30.172.228:8080,172.30.234.176:8080,172.30.234.177:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Kubernetes added a NodePort, in the example with port value `32387`.

You can now connect to the service from outside the cluster via the public IP address of any worker node in the cluster and traffic will be forwarded to the service. Service discovery with the selector and labels is used to deliver the request to one of the pod's IP addresses. With this piece in place we now have a complete pipeline for load balancing external client requests to all the nodes in the cluster.

To connect to the service, we need the Public IP address of one of the worker nodes and the NodePort of the Service. You can use a bash processor called [`jq`](https://stedolan.github.io/jq/) to parse JSON from command line.

```console
PUBLIC_IP=$(oc get nodes -o wide -o json | jq -r '.items[0].status.addresses | .[] | select( .type=="ExternalIP" ) | .address ')
echo $PUBLIC_IP

NODE_PORT=$(oc get svc helloworld -n $MY_NS --output json | jq -r '.spec.ports[0].nodePort' )
echo $NODE_PORT
```

Test the deployment,

```console
$ curl -L -X POST "http://$PUBLIC_IP:$NODE_PORT/api/messages" -H 'Content-Type: application/json' -H 'Content-Type: text/plain' -d '{ "sender": "world1" }'

{"id":"f142f74f-c679-4738-96e3-6518e607efa2","sender":"world1","message":"Hello world1 (direct)","host":null}
```

The client connects to your application via a public IP address of a worker node and the NodePort. Each node proxies the port, `kube-proxy` receives the request, and forwards it to the service at the cluster IP. At this point the request matches the netfilter or `iptables` rules and gets redirected to the server pod. 

However, we still require some level of load balancing. a `LoadBalancer` service is the standard way to expose a service.

## Next

Go to [LoadBalancer](loadbalancer.md) to learn more about ServiceType LoadBalancer.
