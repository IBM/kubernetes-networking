# Bring your own Ingress controller

run Ingress on a free 1 node IKS cluster

In IBM Cloud Kubernetes Service, IBM-provided application load balancers (ALBs) are based on a custom implementation of the NGINX Ingress controller. However, depending on what your app requires, you might want to configure your own custom Ingress controller instead of using the IBM-provided ALBs. For example, you might want to use the Istio ingressgateway load balancer service to control traffic for your cluster. When you bring your own Ingress controller, you are responsible for supplying the controller image, maintaining the controller, updating the controller, and any security-related updates to keep your Ingress controller free from vulnerabilities.

## Expose your Ingress controller by creating an NLB and a hostname

1. Create a free cluster
1. Login to your cluster
1. List existing NLBs

```
CLUSTERNAME=remkohdev-iks116-1n-cluster-free
% ibmcloud ks nlb-dns ls -c $CLUSTERNAME 
OK
Subdomain   Load Balancer Hostname   SSL Cert Status   SSL Cert Secret Name   Secret Namespace
```

```
kubectl config current-context
ibmcloud ks worker ls --cluster $CLUSTERNAME
OK
ID                                                       Public IP        Private IP      Flavor   State    Status   Zone    Version   
kube-br2lrckd0icvsu163bt0-remkohdevik-default-000000f5   173.193.106.26   10.76.114.251   free     normal   Ready    hou02   1.16.9_1531

% ibmcloud ks zones --provider classic 
% ibmcloud ks vlan ls --zone                         
OK
ID   Name   Number   Type   Router   Supports Virtual Workers 

ibmcloud ks cluster get --cluster $CLUSTERNAME
```


```
% ibmcloud ks worker-pool ls --cluster $CLUSTERNAME
OK
Name      ID                             Flavor   Workers   
default   br1vf5df00av3tu5nbd0-45ebec5   free     1   

```

1. Create a Network Load Balancer (NLB) to expose your custom Ingress controller deployment by creating a LoadBalancer service,

```
echo 'apiVersion: v1
kind: Service
metadata:
  name: my-ingress
  labels:
    app: my-ingress
spec:
  ports:
  - port: 80
  selector:
    app: my-ingress
  type: LoadBalancer' > my-ingress.yaml
```

```
kubectl create -f my-ingress.yaml
```

apiVersion: v1
kind: Service
metadata:
  name: my-ingress
  annotations:
    service.kubernetes.io/ibm-load-balancer-cloud-provider-ip-type: <public_or_private>
    service.kubernetes.io/ibm-load-balancer-cloud-provider-vlan: "<vlan_id>"
spec:
  type: LoadBalancer
  selector:
    <selector_key>: <selector_value>
  ports:
   - protocol: TCP
     port: 8080
  loadBalancerIP: <IP_address>

and then create a hostname for the NLB IP address.

Get the configuration file for your Ingress controller ready. For example, you can use the cloud-generic NGINX community Ingress controller. If you use the community controller, edit the mandatory.yaml file by following these steps.

Replace all instances of namespace: ingress-nginx with namespace: kube-system.
Replace all instances of the app.kubernetes.io/name: ingress-nginx and app.kubernetes.io/part-of: ingress-nginx labels with one app: ingress-nginx label.

Deploy your own Ingress controller. For example, to use the cloud-generic NGINX community Ingress controller, run the following command.

```
kubectl apply -f mandatory.yaml -n kube-system
```

Define a load balancer service to expose your custom Ingress deployment.

```
apiVersion: v1
kind: Service
metadata:
    name: ingress-nginx
spec:
    type: LoadBalancer
    selector:
        app: ingress-nginx
    ports:
    - name: http
        protocol: TCP
        port: 80
    - name: https
        protocol: TCP
        port: 443
    externalTrafficPolicy: Cluster
```

Create the service in your cluster.

```
kubectl apply -f ingress-nginx.yaml -n kube-system
```

Get the EXTERNAL-IP address for the load balancer.

```
kubectl get svc ingress-nginx -n kube-system
```
