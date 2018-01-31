# Deploying Log Aggregation

In this lab you will learn how to deploy log aggregation. Deployment of logging needs, similar to the [metrics](deploying_metrics.md) lab, a backend storage. This lab will leverage dynamic PV creation to provide this storage.

## Step 1

Switch to the `logging` project

```
oc project logging
```

Using the ansible playbook provided by OpenShift, deploy the logging stack and make sure these options are set to what makes sense to you.

```
ansible-playbook -i <your-inventory-file> \
/usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/openshift-logging.yml \
-e openshift_logging_install_logging=true \
-e openshift_logging_use_ops=true \
-e openshift_logging_install_eventrouter=true \
-e openshift_logging_purge_logging=true \
-e openshift_logging_es_cpu_limit=1000m \
-e openshift_logging_es_memory_limit=1Gi \
-e openshift_logging_es_pvc_dynamic=true \
-e openshift_logging_es_pvc_storage_class_name=gp2 \
-e openshift_logging_es_pvc_size=6Gi \
-e openshift_logging_es_ops_pvc_dynamic=true \
-e openshift_logging_es_ops_pvc_storage_class_name=gp2 \
-e openshift_logging_es_ops_pvc_size=4Gi \
-e openshift_logging_es_ops_cpu_limit=1000m \
-e openshift_logging_es_ops_memory_limit=1Gi 

```

There is a known issue where the above doesn't always work and you have to manually set this up in the `dc`

First find the name of your `dc`

```
$ oc get dc --selector logging-infra=elasticsearch -o name
deploymentconfigs/logging-es-data-master-wrlmbsgn
```

Overwrite the `emptyDir` with the right `pvc`
```
$  oc volume dc/logging-es-data-master-wrlmbsgn --add --overwrite --name=elasticsearch-storage -t pvc --claim-name=logging-storage
deploymentconfig "logging-es-data-master-wrlmbsgn" updated
```

Verify with
```
$ oc  get dc/logging-es-data-master-wrlmbsgn -o yaml | grep -B2 logging-storage
      - name: elasticsearch-storage
        persistentVolumeClaim:
          claimName: logging-storage

$ oc get pvc
NAME              STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS    AGE
logging-storage   Bound     pvc-4fa4be3a-c415-11e7-8f74-029fe14a0ff8   20Gi       RWO           gluster-block   16m

```

## Conclusion

In this lab you learned how to set up logging with a backend storage. You should now have the logging stack  running.

```
$ oc get pods
NAME                                      READY     STATUS    RESTARTS   AGE
logging-curator-1-65b4r                   1/1       Running   0          10m
logging-es-data-master-wrlmbsgn-2-sj9vh   1/1       Running   0          7m
logging-fluentd-4x2q2                     1/1       Running   0          10m
logging-fluentd-d16p9                     1/1       Running   0          10m
logging-fluentd-jxws9                     1/1       Running   0          10m
logging-kibana-1-t4jvr                    2/2       Running   0          10m
```
