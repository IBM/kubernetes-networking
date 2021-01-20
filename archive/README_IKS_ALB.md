# Lab05 - IKS provided ALB
    
1. Check the Default ALBs
    * Login to the IKS Cluster

        ```console
        $ ibmcloud login -a cloud.ibm.com -r us-south -g default
        $ ibmcloud ks cluster-config --cluster <cluster-name>
        ```

    * Query the ALBs,

        ```
        % ibmcloud ks albs --cluster remkohdev-iks116-3x-cluster
        OK
        ALB ID    Enabled   Status    Type    ALB IP    Zone    Build    ALB VLAN ID   NLB Version   
        private-crbqfo73dd0ajmhsng0e5g-alb1   false    disabled   private   -    dal10   ingress:/ingress-auth:    2832804    1.0
        public-crbqfo73dd0ajmhsng0e5g-alb1    true    enabled    public    169.47.196.187    dal10   ingress:627/ingress-auth:401   2832802    1.0
        ```

        * The private ALB is disabled, the public ALB is enabled.

2. Deploy the Guestbook v1 Application

        ```console
        kubectl create -f redis-master-deployment.yaml
        kubectl create -f redis-master-service.yaml
        kubectl create -f redis-slave-deployment.yaml
        kubectl create -f redis-slave-service.yaml
        kubectl create -f guestbook-deployment.yaml
        kubectl create -f guestbook-service.yaml
        ```

    * Get cluster information

        ```console
        $ ibmcloud ks cluster-get <cluster-name>
        ```

    * Or use grep to filter

        ```console
        $ ibmcloud ks cluster-get <cluster-name> | grep Ingress
        Ingress Subdomain:              <cluster-name>.us-south.containers.appdomain.cloud   
        Ingress Secret:                 <cluster-name>  
        ```

    * Access the Guestbook application via the IKS provided public ALB,

        ```console
        $ open http://<cluster-name>.us-south.containers.appdomain.cloud:31724/
        ```
