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
systemctl restart atomic-openshift-master.service 
```

Verify that it is running

```
systemctl status -l atomic-openshift-master.service 
● atomic-openshift-master.service - Atomic OpenShift Master
   Loaded: loaded (/usr/lib/systemd/system/atomic-openshift-master.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2017-07-24 14:57:20 PDT; 30s ago
     Docs: https://github.com/openshift/origin
 Main PID: 46889 (openshift)
   Memory: 275.9M
   CGroup: /system.slice/atomic-openshift-master.service
           └─46889 /usr/bin/openshift start master --config=/etc/origin/master/master-config.yaml --loglevel=2

Jul 24 14:57:46 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:46.236682   46889 panics.go:76] GET /api/v1/namespaces/myproject/secrets/default-dockercfg-pr4b5: (1.646163ms) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.11:40302]
Jul 24 14:57:46 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:46.810129   46889 panics.go:76] GET /api/v1/namespaces/default/secrets/router-token-4knlh: (1.969476ms) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.10:34418]
Jul 24 14:57:46 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:46.810230   46889 panics.go:76] GET /api/v1/namespaces/default/secrets/router-certs: (1.622118ms) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.10:34418]
Jul 24 14:57:47 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:47.076398   46889 panics.go:76] GET /api/v1/namespaces/default/secrets/router-dockercfg-7wzg6: (1.44486ms) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.10:34418]
Jul 24 14:57:47 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:47.855186   46889 panics.go:76] GET /api/v1/nodes?resourceVersion=0: (773.971µs) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.10:34416]
Jul 24 14:57:49 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:49.977809   46889 panics.go:76] GET /api/v1/namespaces/limits-quotas/secrets/default-token-f8gwn: (1.267002ms) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.11:40302]
Jul 24 14:57:50 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:50.236576   46889 panics.go:76] GET /api/v1/namespaces/limits-quotas/secrets/default-dockercfg-qr9mr: (1.708641ms) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.11:40302]
Jul 24 14:57:51 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:51.294091   46889 panics.go:76] GET /api/v1/nodes?fieldSelector=metadata.name%3Dmaster.172.16.1.10.nip.io&resourceVersion=0: (694.846µs) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.10:34418]
Jul 24 14:57:51 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:51.297765   46889 panics.go:76] GET /apis/extensions/v1beta1/thirdpartyresources: (1.078497ms) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.10:34416]
Jul 24 14:57:51 master.172.16.1.10.nip.io atomic-openshift-master[46889]: I0724 14:57:51.303477   46889 panics.go:76] PUT /api/v1/nodes/master.172.16.1.10.nip.io/status: (5.325891ms) 200 [[openshift/v1.5.2+43a9be4 (linux/amd64) kubernetes/43a9be4] 172.16.1.10:34418]
```

## Step 3

On the webui, login as `user-1` and create a new project

![image](images/new-project-template.png)

Under the overview page; click on `Resources ~> Quota` and see that the quotas and limits were automatically created.

![image](images/project-template-completed.png)

## Conclusion

In this lab you learned how to edit the master configuration file in order to set the default behavior of project creation. You also learned how to export a configuration to use as a basis of a custom configuration.
