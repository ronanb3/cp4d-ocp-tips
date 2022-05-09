# Prune Db2 databases logs to avoid out of space issue

By default, db2 will save logs and there is no log rotation configured by default. This can lead to out of space issue in the PVC (ODF) or worst in the whole storage (NFS)

Here is the procedure.

Log in to Openshift environment and go to your CP4D project.

```
$ oc login -u apikey -p XXXXXXXX --server=https://c100-e.eu-de.containers.cloud.ibm.com:NNNN
Login successful.

You have access to 69 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "cp4d".

$ oc project cp4d
Already on project "cp4d" on server "https://c100-e.eu-de.containers.cloud.ibm.com:NNNNN"
```

## In CP4D 4.0.3+, let's turn auto prune archieve_log

```sh
oc rsh c-db2u-dv-db2u-0 bash

su - db2inst1

bigsql_version=$(cat /opt/dv/current/version | cut -d'=' -f2)

/usr/ibmpacks/bigsql/${bigsql_version}/bigsql/bigsql-cli/BIGSQL/package/scripts/bigsql-auto-logprune.sh -U db2inst1 -M Enable
```

## In CP4D 4.0.2-, let's install a script to remove old db2 logs

Add the following script to a file, say dv-db2-log-cleanup.sh, then run it at the backgroup

```sh
while true; do
  #We need to restrict the file type otherwise find cmd may find Gaiandb db files that must be kept in the pod
  #We keep all the db2diag*.log as they currently have a DIAGSIZE of 300MB
  find ${DIAGPATH} -atime +1 -type f -name "*.log" ! -name "db2diag*.log" -exec ls -l {} \;
  find ${DIAGPATH} -atime +1 -type f -name "*.log" ! -name "db2diag*.log" -exec rm -f {} \;

  find ${DIAGPATH} -atime +1 -type f -name "*.txt" -exec ls -l {} \;
  find ${DIAGPATH} -atime +1 -type f -name "*.txt" -exec rm -f {} \;

  find ${DIAGPATH} -atime +1 -type d -name "FODC*" -exec ls -d {}  \;
  find ${DIAGPATH} -atime +1 -type d -name "FODC*" -exec rm -rf {}  \;
  
  find ${DIAGPATH} -atime +1 -type f -name "core*" -exec ls -l {}  \;
  find ${DIAGPATH} -atime +1 -type f -name "core*" -exec rm -f {}  \;
  sleep 24h #run once every 24 hours
done
```

```sh
oc rsh c-db2u-dv-db2u-0 bash

su - db2inst1

vi dv-db2-log-cleanup.sh

chmod +x dv-db2-log-cleanup.sh; nohup ./dv-db2-log-cleanup.sh &
```

This script can also be used to clean a db2 installation with a full disk, if you can still log in to it long enough.