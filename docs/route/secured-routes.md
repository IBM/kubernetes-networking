# Secured Routes

## Introduction

In OpenShift, there are different types of routes in which you can expose your applications:

* clear,
* edge,
* re-encrypt, and
* pass-through.

The clear route is insecure and doesn't require any certifications, as for the rest of the routes, they are encrypted on different levels and require certificates.

In this tutorial, you will learn how to create 3 types of routes for your applications: clear, edge and passthrough and you will learn the difference in creating each type of route.

## Prerequisites

For this tutorial you will need:

* Sign up for your IBM Cloud account
* Red Hat OpenShift Cluster 4 on IBM Cloud.
* oc CLI (can be downloaded from this link or you can use it at [http://shell.cloud.ibm.com/](http://shell.cloud.ibm.com/).

## Estimated Time

It will take you around 30 minutes to complete this tutorial.

Steps:

* [Login from the CLI & Create Project](https://github.com/nerdingitout/oc-route#login-from-the-cli--create-project)
* [Setting up](https://github.com/nerdingitout/oc-route#setting-up)
* [Create Application](https://github.com/nerdingitout/oc-route#create-application)
* [Expose the Route](https://github.com/nerdingitout/oc-route#expose-the-route)
* [Extract the SSL Cert Secret](https://github.com/nerdingitout/oc-route#extract-the-ssl-cert-secret)
* [Create Edge Route](https://github.com/nerdingitout/oc-route#create-edge-route)
* [Create Golang Application](https://github.com/nerdingitout/oc-route/blob/main/README.md#create-golang-application)
* [Create Passthrough Route](https://github.com/nerdingitout/oc-route#create-passthrough-route)

## Login from the CLI & Create Project

* Go to the web console and click on your username at the top right then 'Copy Login Command', then display the token and copy the ```oc login``` command in your terminal.

![login](https://user-images.githubusercontent.com/36239840/97104809-26821500-16d0-11eb-936e-c2b7fb914523.JPG)

* Create ```my-route-project``` project.

```bash
oc new-project my-route-project
```

## Setting up

In this section, you will view details about your OpenShift Cluster on IBM Cloud. Details you would be interested in are hostname, SSL Cert Secret Name, and the namespace that holds the secret.

* First, this is the Overview page that shows some details about your cluster like Cluster ID and resource group they can be useful when using ibmcloud CLI to grab some information about your cluster.

![image](https://user-images.githubusercontent.com/36239840/113106747-2b656a80-9214-11eb-82ef-4cc6efcd967b.png)

* on the top right, click on your profile and select "Log in to CLI and API"

![image](https://user-images.githubusercontent.com/36239840/113106901-551e9180-9214-11eb-8d61-15445a3b7215.png)

* Copy the IBM Cloud CLI login command and paste it in your CLI

![image](https://user-images.githubusercontent.com/36239840/113106986-6b2c5200-9214-11eb-857f-46d7dcdd266d.png)

![image](https://user-images.githubusercontent.com/36239840/113107025-75e6e700-9214-11eb-8a95-a04005b3002a.png)

* Select the resource group where your cluster resides (in my case it is default)

```bash
ibmcloud target -g default
```

![image](https://user-images.githubusercontent.com/36239840/113107089-8ac37a80-9214-11eb-948a-4b1548b5df81.png)

* View information about your cluster

```bash
ibmcloud oc nlb-dns ls --cluster <cluster_name_or_id>
```

![image (18)](https://user-images.githubusercontent.com/36239840/113107233-bcd4dc80-9214-11eb-8eae-25c4b6a2c201.png)

## Create Application

* Create a new deployment resource using the [ibmcom/guestbook:v2](https://hub.docker.com/r/ibmcom/guestbook/tags) docker image in the project we just created.

```bash
oc new-app myguestbook --image=ibmcom/guestbook:v1
```

* This deployment creates the corresponding Pod that's in running state. Use the following command to see the list of pods in your namespace.

```bash
oc get pods
```

![get pods](https://user-images.githubusercontent.com/36239840/97298061-5534f280-186c-11eb-9dbb-982f2f1c20e0.JPG)

* reate a Kubernetes ClusterIP service for your app deployment. The service provides an internal IP address for the app that the router can send traffic to.

```bash
oc expose deployment myguestbook --type="NodePort" --port=3000
```

## Expose the route

* To view the service that we need to expose. Use the following command.

```bash
oc get svc
```

![image](https://user-images.githubusercontent.com/36239840/113107705-484e6d80-9215-11eb-8b4c-3ff6ff9f0895.png)

* Notice that the application isn't accessible externally, we have only exposed the deployment internally, to make it externally accessible use the following command

```bash
oc expose svc myguestbook 
```

* Now to get the route using the following command

```bash
oc get routes
```

![image (20)](https://user-images.githubusercontent.com/36239840/113109494-1c33ec00-9217-11eb-8c96-675a6efdff35.png)

* copy and paste the link in your browser, you will be redirected to a web page like the following screenshot. Notice that the webpage is not secure because we haven't used any type of encryption yet.
![image](https://user-images.githubusercontent.com/36239840/113109564-281fae00-9217-11eb-843c-6a01474ace7a.png)

* Since you will be using the same application to create an edge route, make sure to delete the route before moving to the next step

```bash
oc delete route myguestbook
```

## Extract the SSL Cert Secret

* Now let's take a look at the secrets in openshift-ingress project. You will need a TLS secret that's generated for your cluster which is of type kubernetes.io/tls.

```bash
oc get secrets -n openshift-ingress
```

![image](https://user-images.githubusercontent.com/36239840/113153878-8e70f480-9248-11eb-843c-5884a84cd398.png)

* View the secret values in your command line, notice that the key and certificate pair are saved in PEM encoded files.

```bash
oc extract secret/<YOUR-TLS-SECRET-NAME> --to * -n openshift-ingress
```

![image](https://user-images.githubusercontent.com/36239840/113154158-c8da9180-9248-11eb-8cb5-acf2c3bd3423.png)

* Save the secret in a temporary directory

```bash
oc extract secret/<YOUR-TLS-SECRET-NAME> --to=/tmp -n openshift-ingress
```

![image](https://user-images.githubusercontent.com/36239840/113154290-f0315e80-9248-11eb-843f-18bc881c6d12.png)

## Create Edge Route

* Create the edge route using the following command

```bash
oc create route edge --service myguestbook --key /tmp/tls.key --cert /tmp/tls.crt
```

* Get the route of your application and open it from your browser

```bash
oc get routes
```

* The application has been deployed successfully

![image](https://user-images.githubusercontent.com/36239840/113154769-68981f80-9249-11eb-965d-3a2f6048df46.png)

* You can check information about the secured website and certificate from the lock icon at the top left of the browser

![image](https://user-images.githubusercontent.com/36239840/113154821-764da500-9249-11eb-8916-559c5e4fd575.png)

![image](https://user-images.githubusercontent.com/36239840/113154850-7c438600-9249-11eb-8b58-2bc906abac9d.png)

## Create Golang Application

In this section, you will be deploying a new application that you will be using for both passthrough and re-encrypt routes, then you will create a secret and mount it to the volume so you can create the routes.

* Create the deployment config and service using oc create command.

```bash
oc create -f https://raw.githubusercontent.com/nerdingitout/oc-route/main/golang-https.yml
```

![image](https://user-images.githubusercontent.com/36239840/113155813-7bf7ba80-924a-11eb-81b0-1b574e925707.png)

* Create TLS secret using the same secret you extracted earlier.

```bash
oc create secret tls mycert --cert /tmp/tls.crt --key /tmp/tls.key
```

![image](https://user-images.githubusercontent.com/36239840/113155891-903bb780-924a-11eb-911e-573ca4289450.png)

* Mount the secret to your volume.

```bash
oc set volume dc/golang-https --add -t secret -m /go/src/app/certs --name cert --secret-name mycert
```

## Create Passthrough Route

* Create the passthrough route

```bash
oc create route passthrough golang-https --service golang-https
```

* Get the URL

```bash
oc get routes
```

* Access the application and view the certificate
![image](https://user-images.githubusercontent.com/36239840/113156191-d264f900-924a-11eb-8565-70d79731a61f.png)

## Resources

You can learn more using the following resources:

* [OpenShift Routes on IBM Cloud](https://cloud.ibm.com/docs/openshift?topic=openshift-openshift_routes)
* [OpenSSL](https://www.digicert.com/kb/ssl-support/openssl-quick-reference-guide.htm)
* [Self-Serviced End-to-end Encryption Approaches for Applications Deployed in OpenShift](https://www.openshift.com/blog/self-serviced-end-to-end-encryption-approaches-for-applications-deployed-in-openshift)
* [Secured Routes](https://docs.openshift.com/container-platform/4.5/networking/routes/secured-routes.html)
* [End to End Encryption with OpenShift: Part 1](https://developers.redhat.com/blog/2017/01/24/end-to-end-encryption-with-openshift-part-1-two-way-ssl/)
* [End to End Encryption with OpenShift: Part 2](https://www.openshift.com/blog/self-serviced-end-to-end-encryption-for-kubernetes-applications-part-2-a-practical-example)
