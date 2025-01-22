# OpenShift APIs for Data Protection (OADP) for Red Hat OpenShift Data Foundation MultiCloud Gateway Database

Taking a backup of your MCG Database
```
export STORAGE=<your OADP storage location>

envsubst < backup.yml | oc -n openshift-adp create -f -
``` 

Restoring the backup of your MCG Database

```
oc -n openshift-storage scale --replicas=0 deploy/odf-operator-controller-manager statefulset/noobaa-db-pg
oc -n openshift-storage delete pvc -l noobaa-db=postgres statefulset/noobaa-db-pg
``` 

After all active connections to the Database have been terminated and all MCG/noobaa related pods are down you can initiate the Restore.

```
export BACKUPID=$(oc -n openshift-adp get backup.velero.io  -l app=openshift-storage -o name | cut -f2 -d'/')
envsubst < restore.yml | oc -n openshift-adp create -f -
```

Due to the duration of the Database restore (depending on your Database size) the Endpoint and Core MCG pods might be already up before the 
Database was active and serving correct content. 
Ensure to reconcile the pods accordingly after the Database restore has finished.

Verify functionality through following command

```
$ oc -n openshift-storage exec -ti noobaa-db-pg-0 -c db -- psql -U postgres -d nbcore -c "SELECT count(*) FROM accounts;"
 count
-------
    16
(1 row)
```
