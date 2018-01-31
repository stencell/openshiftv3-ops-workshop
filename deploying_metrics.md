# Deploying Metrics

In this lab you will learn how to deploy metrics. Deployment of metrics need a backend storage for this. Please see either the [Persistant Volume Claim](creating_persistent_volume.md) or the [Container Native Storage](cns.md) labs before doing this lab.

## Step 1

Switch to the `openshift-infra` project

```
oc project openshift-infra
```

Using the ansible playbook provided by OpenShift, deploy the metrics stack and make you change these options to what makes sense to you. Take note of the options `openshift_metrics_cassandra_storage_type` and `openshift_metrics_cassandra_pvc_size`.

```
ansible-playbook -i <your-inventory-file> \
/usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/openshift-metrics.yml \
-e openshift_metrics_install_metrics=true \
-e openshift_metrics_cassandra_pvc_size=5G \
-e openshift_metrics_cassandra_storage_type=dynamic \
-e openshift_metrics_cassandra_limits_cpu=1500m \
-e openshift_metrics_cassandra_requests_cpu=500m \
-e openshift_metrics_cassandra_limits_memory=2Gi \
-e openshift_metrics_cassandra_requests_memory=500Mi
```

## Conclusion

In this lab you learned how to set up metrics with a backend storage. You should now have metrics collection running.

```
$ oc get pods
NAME                         READY     STATUS    RESTARTS   AGE
hawkular-cassandra-1-08jnc   1/1       Running   0          2m
hawkular-metrics-19vs0       1/1       Running   0          7m
heapster-90862               1/1       Running   0          7m
```
