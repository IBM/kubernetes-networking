# Setup

## Pre-requisities

* Free IBM Cloud account, to create a new IBM Cloud account go [here](https://ibm.github.io/workshop-setup/NEWACCOUNT/).
* Free `Pay-As-You-Go` account. To upgrade a free IBM Cloud account, go [here](https://ibm.github.io/workshop-setup/PAYASYOUGO/).
* [Log in to your IBM Cloud account](https://ibm.github.io/workshop-setup/IBMCLOUD/),

## Setup CLI

In this tutorial, we use the [IBM Cloud Shell](https://cloud.ibm.com/shell), a cloud managed instance of a client terminal with pre-installed tools like ibmcloud CLI and the OpenShift CLI.

[Log into your IBM Cloud account](https://ibm.github.io/workshop-setup/IBMCLOUD/) and open the [IBM Cloud Shell](https://cloud.ibm.com/shell).

The VPC infrastructure service plug-in to the IBM Cloud CLI is already installed in the IBM Cloud Shell.

## Login to IBM Cloud

If you are using the IBM Cloud Shell in your IBM Cloud account, you will already be logged in to your account in the CLI as well.

Check your account settings,

```bash
ibmcloud target
```

List available regions,

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

Target a region as the default location,

```bash
MY_REGION=us-south
ibmcloud target -r $MY_REGION
```

Create a new resource group or use an existing resource group,

```bash
export MY_RG=<username>-vpc-rg
ibmcloud resource group-create $MY_RG
ibmcloud target -g $MY_RG
```

## Next

Next, [create a VPC](2_create_vpc.md).
