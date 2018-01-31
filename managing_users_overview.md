# Managing Users Overview

In this lab you will learn how OpenShift manages users and how authentication is handled. You will also learn how to add/delete users from the OpenShift platform.

## Step 1

OpenShift takes "hands off" approach in regaurds to users. That is, OpenShift does not manage users directly. Instead it "offloads" the user administration to whatever mechanism you choose.

Currently supported authentication methods are:

* LDAP
* HTPassword File
* Github
* Gitlab
* OpenID
* Google
* Request Header

By default, openshift uses `htpasswd` and a "flat file" for authentication. Take a look at this configuration.

```
grep -A4 "htpasswd_auth" /etc/origin/master/master-config.yaml
    name: "htpasswd_auth"
    provider:
      apiVersion: v1
      file: /etc/origin/master/htpasswd
      kind: HTPasswdPasswordIdentityProvider
```

This configuration says that when a users tires to login; send the request to this flat file. Take a look at the users currently on the system.

```
oc get users
NAME      UID                                    FULL NAME       IDENTITIES
demo      72455092-6dad-11e7-b505-5254005e6599                   Local Authentication:demo
```

## Step 2

To add a user into OpenShift; just add the user to the backend authentication system. In this case; we use the `htpasswd` command to update the flat file.

```
htpasswd /etc/origin/master/htpasswd testuser
New password: 
Re-type new password: 
Adding password for user testuser

cat /etc/origin/master/htpasswd 
demo:$apr1$tuMj6pjc$uHo8IoNUoK0mGx6omnI1l1
testuser:$apr1$dYwgH8a9$0IjfdOm2DsN7.LvdBRV6F0
```

Now check OpenShift. Note that the user you just created is not there.

```
oc get users
NAME      UID                                    FULL NAME       IDENTITIES
demo      72455092-6dad-11e7-b505-5254005e6599                   Local Authentication:demo
```

Using the webui or the `oc` command; login using this new user. Example

```
oc login -u testuser https://ocp-master-userX-1.openshift.stencell.net
Username: testuser
Password: 
Login successful.

You dont have any projects. You can try to create a new project, by running

    oc new-project <projectname>

oc whoami
testuser
```

Now take a look at your users

```
oc get users
NAME      UID                                    FULL NAME       IDENTITIES
demo      72455092-6dad-11e7-b505-5254005e6599                   Local Authentication:demo
testuser     b40b2c5f-6e47-11e7-b505-5254005e6599                   Local Authentication:testuser

```

What happened? OpenShift creates a corresponding user once a successful authentication happens.

## Step 3

Now we will go through the steps of deleting a user. First step is to remove/lock the user account in the backend authenication method. In this case, it is as easy as removing them from the file.

```
sed -i '/^user1/d' /etc/origin/master/htpasswd

cat /etc/origin/master/htpasswd
demo:$apr1$tuMj6pjc$uHo8IoNUoK0mGx6omnI1l1
```

Take a look at the users

```
oc get users
NAME      UID                                    FULL NAME       IDENTITIES
demo      72455092-6dad-11e7-b505-5254005e6599                   Local Authentication:demo
testuser     b40b2c5f-6e47-11e7-b505-5254005e6599                   Local Authentication:user
```

OpenShift does not know about what is going on in the backend authentication system. The user would simply not be able to login


```
oc login -u testuser https://ocp-master-userX-1.openshift.stencell.net
Username: testuser
Password:  
Login failed (401 Unauthorized)
```

Once you have locked/deleted a user from the backend authentication system. Just simply delete the user

```
oc delete user testuser
user "testuser" deleted
```

You should now see the user gone from the list

```
oc get users
NAME      UID                                    FULL NAME       IDENTITIES
demo      72455092-6dad-11e7-b505-5254005e6599                   Local Authentication:demo
```

**CLEANUP:** If this user was an admin/owner of any projects; those projects would still exist. You just need to assign them to different users.


## Conclusion

In this lab you learned how users are managed inside of OpenShift. You also go familiar with authentication and how that is handled in OpenShift
