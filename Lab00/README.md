# Setup

If you participate in an IBM managed workshop, you can access a cluster for the workshop via your IBM Cloud account and IBM Id. Instructions are found [here](https://github.com/remkohdev/kube101/tree/master/Lab0).

I recommended using the IBM Cloud shell with tools pre-installed to run the labs at https://shell.cloud.ibm.com.

1. From the Cloud shell, clone the guestbook application,

    ```
    $ git clone https://github.com/IBM/guestbook.git
    $ ls -al
    ```

2. Log in to your cluster, e.g. if created in the `us-south` region,

    ```
    $ ibmcloud login -a cloud.ibm.com -r us-south
    ```

3. If you are using federated SSO login, use the `-sso` flag instead.
4. Select the account in which the cluster was created.

![Login to IBM Cloud](../images/shell-login-to-cloud.png)

1. Download the cluster configuration to the client,

    ```
    $ CLUSTER_NAME=<clustername>
    $ ibmcloud ks cluster config --cluster $CLUSTER_NAME
    $ kubectl config current-context
    ```
