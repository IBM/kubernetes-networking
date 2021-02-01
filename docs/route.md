# Route

Services of type `LoadBalancer` cannot do TLS termination, virtual hosts or path-based routing. These limitations led to the addition in Kubernetes v1.2 of a separate kubernetes resource called `Ingress` and `Route` (on OpenShift).

OpenShift's `Route` was created for the same purpose as the Kubernetes `Ingress` resource, with a few additional capabilities such as splitting traffic between multiple backends, and sticky sessions. In a [Red Hat OpenShift Kubernetes Service (ROKS) cluster](https://cloud.ibm.com/docs/openshift?topic=openshift-ingress-about-roks4), the router is a layer 7 load balancer which implements an [`HAProxy` Ingress controller](https://github.com/haproxytech/kubernetes-ingress). When a Route is created, the built-in `HAProxy` load balancer picks it up in order to expose the requested service.

The `RouteSpec` for a `Route` describes the hostname or path the route exposes, security information, and one to four services the route points to. Requests are distributed among the backends depending on the weights assigned to each backend. Weights are between 0 and 256 with default 100. The tls field is optional and allows specific certificates or behavior for the route.

A router uses the service selector to find the service and the endpoints backing the service. Routes can be sharded among the set of routers. Each router in the group serves only a subset of traffic.

OpenShift defines four types of Routes based on the type of TLS termination that your app requires:

1. Simple for non-encrypted HTTP traffic, require no key or certificates.
2. Passthrough when no TLS termination for encrypted HTTPS traffic is needed, TLS termination is done by the app,
3. Edge, TLS connection between the client and the router service is terminated, and the connection between the router service and your app pod is unencrypted.
4. Re-encrypt, TLS connection between the client and the router service is terminated, and a new TLS connection between the router service and your app pod is created.



## Create a Route

Expose the `helloworld` service as a Route by using the `oc expose` command.

```
$ oc expose service helloworld -n $MY_NS

route.route.openshift.io/helloworld exposed
```

Get all routes,

```
$ oc get routes -n $MY_NS

NAME    HOST/PORT    PATH    SERVICES    PORT    TERMINATION    WILDCARD
helloworld   helloworld-my-apps.remkohdev-roks45-2n-clu-2bef1f4b4097001da9502000c44fc2b2-0000.us-south.containers.appdomain.cloud    helloworld    http-server    None
```

Retrieve the created host for the Route and the NodePort of the `helloworld` service,

```
ROUTE=$(oc get routes -n $MY_NS -o json | jq -r '.items[0].spec.host')
echo $ROUTE

NODE_PORT=$(oc get svc helloworld -n $MY_NS --output json | jq -r '.spec.ports[0].nodePort' )
echo $NODE_PORT
```

Send a request to the Route host for your Service,

```
$ curl -L -X POST "http://$ROUTE:$NODE_PORT/api/messages" -H 'Content-Type: application/json' -d '{ "sender": "world5" }'

{"id":"ba8b9101-6594-420b-8cab-251dc7a2de65","sender":"world5","message":"Hello world5 (direct)","host":null}
```

## Next

Next, go to [Network Policy](networkpolicy.md).