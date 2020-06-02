# Lab04 - Extra

I want to deploy two versions of `helloworld`, a direct `helloworld` and a `helloworld-proxy`, which is a proxy to the `helloworld` application. To access the direct version, I add a path `/helloworld`. To access the proxy version, I add a path `/helloworld/proxy`. 

First deploy the proxy.

```
$ kubectl create -f helloworld-proxy-deployment.yaml
$ kubectl create -f helloworld-proxy-service-loadbalancer.yaml
```

Get the proxy service details and test the proxy,

```
$ kubectl get svc helloworld-proxy
NAME    TYPE    CLUSTER-IP    EXTERNAL-IP    PORT(S)    AGE
helloworld-proxy    LoadBalancer    172.21.78.158    169.48.67.164    8080:30940/TCP    61s
```

To test use the External-IP and the NodePort, e.g. 169.48.67.164:30940, and send the proxy the Kubernetes service name and target port,

```
$ $ curl -L -X POST "http://169.48.67.164:30940/proxy/api/messages" -H 'Content-Type: application/json' -H 'Content-Type: application/json' -d '{ "sender": "remko", "host": "helloworld-proxy:8080" }'

{"id":"3aa4f889-94a0-4be8-b8f2-8b59f8ad3de7","sender":"remko","message":"Hello remko (proxy)","host":"helloworld-proxy:8080"}
```

The proxy will add a `proxy` message to the created message.

You need the Ingress Subdomain and Ingress Secret of your cluster to configure your Ingress resource. 


## Resources

- See: https://kubernetes.io/docs/concepts/services-networking/ingress/
- https://medium.com/google-cloud/understanding-kubernetes-networking-ingress-1bc341c84078
- https://github.com/ibm-cloud-docs/containers/blob/master/cs_ingress.md
- [Kong, "the world's most popular open source API Gateway"](https://konghq.com/kong/), [tutorial with Kong Ingress](https://medium.com/swlh/kubernetes-ingress-simplified-e0b9dc32f9fd)