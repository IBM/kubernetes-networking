# Setup

## Pre-requisities

* Free IBM Cloud account, to create a new IBM Cloud account go [here](https://ibm.github.io/workshop-setup/NEWACCOUNT/).
* Free `Pay-As-You-Go` account. To upgrade a free IBM Cloud account, go [here](https://ibm.github.io/workshop-setup/PAYASYOUGO/).
* [Log in to your IBM Cloud account](https://ibm.github.io/workshop-setup/IBMCLOUD/),

## Setup CLI

In this tutorial, we use the IBM Cloud Shell, a cloud managed instance of a client terminal with pre-installed tools like ibmcloud CLI and the OpenShift CLI.

[Log into your IBM Cloud account](https://ibm.github.io/workshop-setup/IBMCLOUD/) and open the [IBM Cloud Shell](https://cloud.ibm.com/shell).

The VPC infrastructure service plug-in to the IBM Cloud CLI is already installed in the IBM Cloud Shell. If you are using an alternative client terminal, install it,

```console
ibmcloud plugin install infrastructure-service
```

You can verify the installed plugins,

```console
$ ibmcloud plugin list

Listing installed plug-ins...

Plugin Name                                  Version   Status             Private endpoints supported   
app-configuration                            1.0.2                        false   
cloud-dns-services                           0.5.0                        true   
cloud-functions[wsk/functions/fn]            1.0.56                       false   
container-registry                           0.1.546   Update Available   true   
vpc-infrastructure[infrastructure-service]   1.5.0     Update Available   true     
```

## Login to IBM Cloud

If you are using the IBM Cloud Shell in your IBM Cloud account, you will already be logged in to your account in the CLI as well.

```bash
ibmcloud target
```

You can target a region as the default location,

```bash
ibmcloud target -r us-south
```

To list available regions,

```bash
$ ibmcloud regions

Name            Display name
au-syd          Sydney
in-che          Chennai
jp-osa          Osaka
jp-tok          Tokyo
kr-seo          Seoul
eu-de           Frankfurt
eu-gb           London
ca-tor          Toronto
us-south        Dallas
us-south-test   Dallas Test
us-east         Washington DC
br-sao          Sao Paulo
```

## Next

Next, [Create a VPC](2_create_vpc.md).
