# External Name

## Pre-requisites

Finish the [Services](services.md), [ClusterIP](clusterip.md), [NodePort](nodeport.md), and [LoadBalancer](loadbalancer.md) labs.

## About External Name

An [ExternalName Service](https://kubernetes.io/docs/concepts/services-networking/service/#externalname) is a special case of Service that does not have selectors and uses DNS names instead, e.g.

```yaml
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

## Next

Next,

1. for IKS go to [Ingress](../ingress/ingress-alb.md),
2. for ROKS go to [Route](../route/route.md)..
