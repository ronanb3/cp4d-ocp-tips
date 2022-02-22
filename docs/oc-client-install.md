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

(NB : I changed the content to vois giing up my user id and auth)

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

Source : [Red Hat OpenShify documentation](https://docs.openshift.com/container-platform/4.8/cli_reference/openshift_cli/getting-started-cli.html)

## bash completion

TBD

## How to log to ROKS OpenShift from CLI

ROKS doesn't allow using `oc login` with username and password to log in in OpenShift. Here is the solution to get a `oc login` that works all the time without having to go through the Web interface every day.

