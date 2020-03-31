# CP4D 2.5 installation on TEC

I did all actions from the master server (second server in the list) through ssh. You can connect through the second public IP (10.99.mmm.nnn). Then all installation is done through the internal IP (192.168.1.nnn)

`$ ssh root@10.99.mmm.nnn`

for each prereqs, I test them with the --apply option to check what will be done and then I apply them.

## Downloads

Get installation files 

### For IBMers

```
wget http://icpfs1.svl.ibm.com/zen/cp4d-builds/2.5.0.0/production/installer/63/cpd-linux
wget http://icpfs1.svl.ibm.com/zen/cp4d-builds/2.5.0.0/production/internal/repo.yaml
```

icpfs1 server is IBM internal only, and cannot be reached from TEC servers. Download the files on your PC while connected to IBM VPN and then send them to TEC servers while connected to TEC VPN. 

### For clients

Get CPD installer & API keyfor customers :

from passport advantage (CC3Y1ML) (the bin will download the tgz)

or get the installer direclty :

[https://ibm-open-platform.ibm.com/repos/cpd/v2.5/EE/cloudpak4data-ee-v2.5.0.0.tgz](https://ibm-open-platform.ibm.com/repos/cpd/v2.5/EE/cloudpak4data-ee-v2.5.0.0.tgz)

then get your API key :

[https://myibm.ibm.com/products-services/containerlibrary](https://myibm.ibm.com/products-services/containerlibrary)

## Installation with NFS

I did all installation with NFS as an NFS server comes preconfigured. I don't know the status of Portwaorx at the time I write this documentation and if it will become the new standard.

## Prereqs

```
# chmod a+x ./cpd-linux
# ./cpd-linux adm --repo repo.yaml --assembly lite --namespace zen
# ./cpd-linux adm --repo repo.yaml --assembly lite --namespace zen —apply
```

Zen projects is created by the script

## Installation

```
# oc project zen

Already on project "zen" on server "https://192.168.1.101:8443”.

# ./cpd-linux --repo repo.yaml --assembly lite --namespace zen -c nfs-client --transfer-image-to=docker-registry.default.svc:5000/zen --target-registry-username=$(oc whoami) --target-registry-password=$(oc whoami -t) --accept-all-licenses
```

Everything should run fine

You can access CP4D at [https://zen-cpd-zen.apps.tec.uk.ibm.com/](https://zen-cpd-zen.apps.tec.uk.ibm.com/)

The default login is admin / password

PS : don't forget to connect to the VPN to get access, I often miss this part and wonder why my installation is down ;)

CP4D 2.5 installation is minimal, even Watson Studio is not installed

## Watson Studio

Source : [https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/install/install-ws.html](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/install/install-ws.html)

So now let’s install Watson Studio

Set the parameter on every node in the cluster:
echo "vm.max_map_count=262144" >> /etc/sysctl.conf ; sysctl -p

Set the time zone for the master node

```
# vi override.yaml

global:
    masterTimezone: 'Europe/Paris'
```

### Prereqs

```
# ./cpd-linux adm --repo repo.yaml  --assembly wsl --namespace zen --apply
# ./cpd-linux adm --repo repo.yaml  --assembly wsl --namespace zen
```

### Installation

```
# ./cpd-linux --repo repo.yaml --assembly wsl --namespace zen -c nfs-client --transfer-image-to=docker-registry.default.svc:5000/zen --target-registry-username=$(oc whoami) --target-registry-password=$(oc whoami -t) --accept-all-licenses —override override.yaml
```

### Spark

Let's install also Spark engine in order to get Spark environment in Studio

```
# ./cpd-linux --repo repo.yaml --assembly spark --namespace zen -c nfs-client --transfer-image-to=docker-registry.default.svc:5000/zen --target-registry-username=$(oc whoami) --target-registry-password=$(oc whoami -t) --accept-all-licenses —override override.yaml
```


You now have a basic CP4D installation. Let's add some addons now.

## Watson Knowledge Catalog

Source : [https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/install/install-wkc.html](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/wsj/install/install-wkc.html)

### Prereqs

Add the following line in /etc/security/limits.conf

```
# vi /etc/security/limits.conf
[…]
*               hard    nofile          66560
*               soft    nofile          66560
```

### Kernel configuration

Update kernel on all server (Masters and workers)

```
# echo "kernel.sem = 250 1024000 32 4096" >> /etc/sysctl.conf ; sysctl -p
# echo "kernel.msgmax = 65536" >> /etc/sysctl.conf ; sysctl -p
# echo "kernel.msgmnb = 65536" >> /etc/sysctl.conf ; sysctl -p
# ipcs -l
```

### Cluster configuration

```
# ./cpd-linux adm --repo repo.yaml  --assembly wkc --namespace zen —apply
# ./cpd-linux adm --repo repo.yaml  --assembly wkc --namespace zen
```

### Installation

```
# ./cpd-linux --repo repo.yaml --assembly wkc --namespace zen -c nfs-client --transfer-image-to=docker-registry.default.svc:5000/zen --target-registry-username=$(oc whoami) --target-registry-password=$(oc whoami -t) --accept-all-licenses --override override.yaml --ask-pull-registry-credentials
```

## Watson OpenScale

Source : [https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/cpd/svc/openscale/openscale-install.html](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/cpd/svc/openscale/openscale-install.html)

### Cluster configuration

```
# ./cpd-linux adm --repo repo.yaml  --assembly aiopenscale --namespace zen —apply
# ./cpd-linux adm --repo repo.yaml  --assembly aiopenscale --namespace zen
```

### Installation

```
# ./cpd-linux --repo repo.yaml --assembly aiopenscale --namespace zen -c nfs-client --transfer-image-to=docker-registry.default.svc:5000/zen --target-registry-username=$(oc whoami) --target-registry-password=$(oc whoami -t) --accept-all-licenses --override override.yaml --ask-pull-registry-credentials
```

## Watson Machine Learning

Source : [https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/cpd/svc/wsj/ml-install.html](https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_current/cpd/svc/wsj/ml-install.html)

```
# ./cpd-linux --repo repo.yaml --assembly wml --namespace zen -c nfs-client --transfer-image-to=docker-registry.default.svc:5000/zen --target-registry-username=$(oc whoami) --target-registry-password=$(oc whoami -t) --accept-all-licenses --override override.yaml --ask-pull-registry-credentials
```

## Tips

Here are some tips on installation and issue solving

### Error initializing source docker...

If you have an issue like "Error initializing source docker://cp.icr.io/cp/cpd/zen-meta-couchdb:v2.5.0.0-210: unable to retrieve auth token: invalid username/password”, check the credentials in repo.yaml.

They change sometimes so refresh the repo.yaml, or better use the myibm key from https://myibm.ibm.com/products-services/containerlibrary. Username is cp in this case.

### Full restart

In some case, debugging a single pod is not enough. I sometimes restart all CP4D with this command to restar all the pods.

`# oc delete pods -n zen --all`

It may take a while to restart all the pods, depending on the number of addons you have installed.


