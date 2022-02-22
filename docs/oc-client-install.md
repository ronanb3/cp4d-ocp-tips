# OpenShift client installation en tips

Here are the steps to install the OpenShift client for Linux and some tips ans tricks to manage connexions

## Install oc client

Navigate to the OpenShift Container Platform downloads page on the Red Hat Customer Portal.

Select the appropriate version in the Version drop-down menu.

Right Click on "Download Now next to the latest OpenShift Linux Client entry to get the link.

Download the client with the link and wget. Don't forget the single quote around the URL to avoind issues.

```sh
wget 'https://access.cdn.redhat.com/content/origin/files/sha256/28/2895de3bb4a9d9a68aa6e48c06ffaed21f81c7341a78e81fddd7c50eaca08c1b/oc-4.9.21-linux.tar.gz?user=c1223590990cfd8b164b125451ffffff&_auth_=1645565566_0d7edf2fef35f7623fa3dd411c310666'
```

Rename the file for easier management

```sh
mv oc-4.9.21-linux.tar.gz\?user\=c1223590990cfd8b164b125451ffffff\&_auth_\=1645565566_0d7edf2fef35f7623fa3dd411c310666 oc-4.9.21-linux.tar.gz
```

(NB : I changed the content to avoid giving up my user id and auth)

Unzip the file

```sh
tar xvzf oc-4.9.21-linux.tar.gz
```

Place the oc binary in a directory that is on your PATH.

```sh
sudo mv oc kubectl /usr/local/bin
```

Check that everything is working

```sh
$ oc version
Client Version: 4.9.21
error: You must be logged in to the server (Unauthorized)
```

The error appears if you are not logged in so don't worry.

Source : [Red Hat OpenShift documentation](https://docs.openshift.com/container-platform/4.8/cli_reference/openshift_cli/getting-started-cli.html)

## bash completion

TBD

## How to log to ROKS OpenShift from CLI with an API key

ROKS doesn't allow using `oc login` with username and password to log in in OpenShift. Here is the solution to get a `oc login` that works all the time without having to go through the Web interface every day.

### Install ibmcloud command

```sh
curl -fsSL https://clis.cloud.ibm.com/install/linux | sh

ibmcloud plugin install container-service

ibmcloud plugin install container-registry

ibmcloud plugin install observe-service
```


### Login to your IBM Cloud

```sh
$ ibmcloud login -sso
API endpoint: https://cloud.ibm.com
Region: eu-de

Get a one-time code from https://identity-2.uk-south.iam.cloud.ibm.com/identity/passcode to proceed.
Open the URL in the default browser? [Y/n] > n
One-time code >
Authenticating...
OK

Select an account:
1. Ronan Bourlier's Account (b02822e1d9cb5263f3d06ffffdcd9793) <-> 1687099
2. WATSON CMCIC (e443898b246db06ab7af7a5ef0498e89)
[...]
Enter a number> 1
Targeted account Ronan Bourlier's Account (b02822e1d9cb5263f3d06ffffdcd9793) <-> 1687099

API endpoint:      https://cloud.ibm.com
Region:            eu-de
User:              ronan.bourlier@fr.ibm.com
Account:           Ronan Bourlier's Account (b02822e1d9cb5263f3d06ffffdcd9793) <-> 1687099
Resource group:    No resource group targeted, use 'ibmcloud target -g RESOURCE_GROUP'
CF API endpoint:
Org:
Space:
```

List your clusters and get the ID your are interested in

```sh
$ ibmcloud oc cluster ls
OK
Name     ID                     State    Created        Workers   Location    Version                 Resource Group Name   Provider
prod01   c7sh01kf0sdsn628u6cg   normal   3 weeks ago    11        Frankfurt   4.8.26_1542_openshift   default               vpc-gen2
prod02   c8ab34rf0s84mif4ffeg   normal   10 hours ago   11        Frankfurt   4.8.26_1542_openshift   default               vpc-gen2
test01   c80pi0mf05u00m1elfgg   normal   2 weeks ago    11        Frankfurt   4.8.26_1542_openshift   default               vpc-gen2
test03   c69spr6f0iqs72cnn5jg   normal   3 months ago   8         Frankfurt   4.8.26_1542_openshift   default               vpc-gen2
test04   c7il1kkf09bln89tt09g   normal   1 month ago    11        Frankfurt   4.8.26_1542_openshift   default               vpc-gen2
```

Get URL for API connect
Select your cluster and get the URL

```sh
export CLUSTER_NAME=prod02
export MASTER_URL=$(ibmcloud oc cluster get -c $CLUSTER_NAME | awk '/^URL:/ {print $2}')
```

Create an API key for your environment

```sh
$ ibmcloud iam api-key-create prod02-ocp
Creating API key prod02-ocp under b02822e1d9cb5263f3d06ffffdcd9793 as ronan.bourlier@fr.ibm.com...
OK
API key prod02-ocp was created

Please preserve the API key! It cannot be retrieved after it's created.

ID            ApiKey-e30537fa-01ab-422f-afbb-68e77dab3bfb
Name          prod02-ocp
Description
Created At    2022-02-22T21:01+0000
API Key       0h6teIifwifCAD61qyXXXX9lQZzQ0RkkNnvYzmaGsTQ7
Locked        false
```

Keep in a safe place the API Key, you cannot retrieve it after. Of course I changed mine in this example ;)

Login with this key in IBM Cloud

```sh
ibmcloud login --apikey 0h6teIifwifCAD61qyXXXX9lQZzQ0RkkNnvYzmaGsTQ7
```

Get context for your cluster

```sh
ibmcloud oc cluster config -c prod02
```

Now you can login with your API key

```sh
oc login -u apikey -p 0h6teIifwifCAD61qyXXXX9lQZzQ0RkkNnvYzmaGsTQ7 --server=https://c100-e.eu-de.containers.cloud.ibm.com:31683
```

I replaced $MASTER_URL by its value to keep this login in a place to use it easily.


