# Setting up Default Limit Ranges and Quotas for Projects

In this lab you will learn how to set it up to where OpenShift applies quotas to newly created projects.

By default OpenShift does not apply any quotas/limits to a newly created project. Instead, it leaves it up to the administrator to apply those quotas/limits. As an OpenShift adminitrator, you can configure OpenShift to set quotas to all newly created projects.

## Step 1

OpenShift uses a default template that creates a project with the requested name, and assigns the requesting user to the "admin" role for that project. You can override this using the `projectRequestTemplate` configuration in the master config file.

Rather than write a complete spec by hand, we will export the existing one.

```
oc adm create-bootstrap-project-template -o yaml > my-project-template.yaml
```

Inspect this file...you will notice items start at about line 6

```
cat my-project-template.yaml -b | head -10
     1	apiVersion: v1
     2	kind: Template
     3	metadata:
     4	  creationTimestamp: null
     5	  name: project-request
     6	objects:
     7	- apiVersion: v1
     8	  kind: Project
     9	  metadata:
    10	    annotations:
```

Here, you can insert the same configuration we used in the [quotas and limits lab](assigning_limit_ranges_and_quotas.md). Go ahead and insert those conifurations in the file.

I put the configuration towards the botton, so the last 40 or so lines should look like this.

```
cat -b my-project-template.yaml | tail -41
    74	- apiVersion: "v1"
    75	  kind: "LimitRange"
    76	  metadata:
    77	    name: "${PROJECT_NAME}-limits"
    78	  spec:
    79	    limits:
    80	      - type: "Pod"
    81	        max:
    82	          cpu: "2"
    83	          memory: "1Gi"
    84	        min:
    85	          cpu: "200m"
    86	          memory: "6Mi"
    87	      - type: "Container"
    88	        max:
    89	          cpu: "2"
    90	          memory: "1Gi"
    91	        min:
    92	          cpu: "100m"
    93	          memory: "4Mi"
    94	        default:
    95	          cpu: "300m"
    96	          memory: "200Mi"
    97	        defaultRequest:
    98	          cpu: "200m"
    99	          memory: "100Mi"
   100	        maxLimitRequestRatio:
   101	          cpu: "10"
   102	- apiVersion: v1
   103	  kind: ResourceQuota
   104	  metadata:
   105	    name: "${PROJECT_NAME}-quota"
   106	  spec:
   107	    hard:
   108	      pods: "4"
   109	parameters:
   110	- name: PROJECT_NAME
   111	- name: PROJECT_DISPLAYNAME
   112	- name: PROJECT_DESCRIPTION
   113	- name: PROJECT_ADMIN_USER
   114	- name: PROJECT_REQUESTING_USER
```

Note I used `"${PROJECT_NAME}-limits"` and `"${PROJECT_NAME}-quota"` for the entry of `metadata.name`. Take your time and make sure you got the spacing right.


Once you are satisfied with what you want the defaults to be; load the template into the default OpenShift namespace.

```
oc create -f my-project-template.yaml -n default
template "project-request" created
```

Make a note of the name of this project template

```
grep -A3 "kind: Template" my-project-template.yaml 
kind: Template
metadata:
  creationTimestamp: null
  name: project-request
```

In our example, it is named "project-request"...you will be needing this for the next step.

## Step 2

Next, you need to tell OpenShift to use this new template by modifying the `/etc/origin/master/master-config.yaml` configuration file.

Make a backup copy of this file (it is sort of an important file)

```
cp /etc/origin/master/master-config.yaml{,.bak}
```

Now under `projectConfig` you need to reference this new default config by editing the config to `projectRequestTemplate: "default/project-request"`. It should look something like this

```
grep -A 5 projectConfig /etc/origin/master/master-config.yaml
projectConfig:
  projectRequestTemplate: "default/project-request"
  defaultNodeSelector: ""
  projectRequestMessage: ""
  projectRequestTemplate: ""
  securityAllocator:
```

Restart the `atomic-openshift-master` service

```
systemctl restart atomic-openshift-master-api.service 
```

Verify that it is running

```
systemctl status -l atomic-openshift-master-api.service 
● atomic-openshift-master-api.service - Atomic OpenShift Master API
   Loaded: loaded (/etc/systemd/system/atomic-openshift-master-api.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2018-01-30 22:54:28 UTC; 21h ago
     Docs: https://github.com/openshift/origin
 Main PID: 2440 (docker-current)
   Memory: 5.4M
   CGroup: /system.slice/atomic-openshift-master-api.service
           └─2440 /usr/bin/docker-current run --rm --privileged --net=host --name atomic-openshift-ma...

Jan 31 20:45:16 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: I0131 20:45:16.133208       1 rest.go:362] Starting watch for /api/v1/services, rv=263116 labels= fields= timeout=8m46s
Jan 31 20:45:19 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: I0131 20:45:19.504759       1 rbac.go:116] RBAC DENY: user "system:anonymous" groups ["system:unauthenticated"] cannot "get" resource "namespaces" in namespace "openshift-metrics"
Jan 31 20:45:19 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: I0131 20:45:19.613418       1 rest.go:362] Starting watch for /api/v1/endpoints, rv=301619 labels= fields= timeout=9m15s
Jan 31 20:45:20 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: I0131 20:45:20.109262       1 rest.go:362] Starting watch for /apis/image.openshift.io/v1/imagestreams, rv=299721 labels= fields= timeout=6m55s
Jan 31 20:45:20 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: E0131 20:45:20.181302       1 watcher.go:210] watch chan error: etcdserver: mvcc: required revision has been compacted
Jan 31 20:45:21 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: I0131 20:45:21.193543       1 rest.go:362] Starting watch for /apis/image.openshift.io/v1/imagestreams, rv=301626 labels= fields= timeout=6m31s
Jan 31 20:45:26 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: I0131 20:45:26.072560       1 rest.go:362] Starting watch for /api/v1/nodes, rv=301621 labels= fields=metadata.name=ip-10-47-3-48.us-west-2.compute.internal timeout=6m6s
Jan 31 20:45:27 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: I0131 20:45:27.646190       1 rest.go:362] Starting watch for /api/v1/namespaces, rv=300395 labels= fields= timeout=6m35s
Jan 31 20:45:27 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: E0131 20:45:27.694945       1 watcher.go:210] watch chan error: etcdserver: mvcc: required revision has been compacted
Jan 31 20:45:28 ip-10-47-3-65.us-west-2.compute.internal atomic-openshift-master-api[2440]: I0131 20:45:28.717870       1 rest.go:362] Starting watch for /api/v1/namespaces, rv=301642 labels= fields= timeout=6m3s
```

## Step 3

On the webui, login as `user-1` and create a new project

![image](images/new-project-template.png)

Under the overview page; click on `Resources ~> Quota` and see that the quotas and limits were automatically created.

![image](images/project-template-completed.png)

## Conclusion

In this lab you learned how to edit the master configuration file in order to set the default behavior of project creation. You also learned how to export a configuration to use as a basis of a custom configuration.
