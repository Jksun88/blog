---
title: "Service Accounts in kubernetes"
datePublished: Mon Sep 26 2022 23:45:20 GMT+0000 (Coordinated Universal Time)
cuid: cl8jf3j3b01y1r0nv2bayemqq
slug: service-accounts-in-kubernetes
tags: security, kubernetes, devops, containers

---

Today I am going to do the summary of the service accounts on kubernetes. 

# What is **service accounts** on kubernetes ? 
Service account is a concept linked to other kubernetes security concept like RBAC (Role Based Access Controls), authorization, authentication and so on. 

There are two types of accounts in kubernetes: 

1. User account : this is the common account provided to human user like a devops, developper or an admin to perform tasks on the cluster. 
2. Service account : this is the account provided to 'machine' an application to interact with a kubernetes cluster. For example we can create a service account for an application who needs to get metrics of the cluster. 

# How do we create a service account ? 
To create a service account we need to use 
```
kubectl create serviceaccount account-name
```
*account-name* is the name you choose for your account (service). 
example: kubectl create serviceaccount rabbit

To list all the service accounts, we use the commonly used syntax for listing on kubernetes 
```
kubectl get serviceaccount
```

When the service account is create, it also create a token. The token will be used by your application while authenticating to kubernetes api.  

The token is store as secret object. 
To check it you can describe the service account. 
```
kubectl describe serviceaccount rabbit
```
check the **tokens** line. 
To view the secret run 
```
kubectl describe secret token-name
```
you will then have the token in clear :))

If the application that want to access the cluster is hosted on the cluster then you can mount the token as a secret volume mount in the pod of the app. 
Each namespace have it's own service account, whenever a pod is created in a namespace, the default token is attached to the pod as a volume mount. 
We will try it with a sample nginx pod just for the test. 
Pod yaml file 

```
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  containers:
    - name: nginx
      image: nginx
``` 
**kubectl -f pod.yaml apply 
kubectl describe pod sample-pod **

![ksnip_20220927-061528.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1664234156763/5vAb82M-q.png align="left")
Check the volume mount of the pod. We can see that secret token is mounted at /var/run/secrets/kubernetes.io/serviceaccounts

We can see three file at that location. The token is contain in the token file. 
We can specify the name of a new service account in the yaml file of the pod. It will look like this: 

```
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  containers:
    - name: nginx
      image: nginx
  serviceAccountName: name
``` 
**Note: **We can't change the service account of an existing pod, we can delete and recreate the pod with the service account we wish to use. But in case of a deployment it will recreate the new pods with the right service account if specified or not. 
If you don't want to mount the service account token automatically on your pods, you can add the 

```
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  containers:
    - name: nginx
      image: nginx
  serviceAccountName: name
  automountServiceAccountToken: false
``` 
# Summary
Service accounts are used to create account to grants access to some applications to run commands on the cluster. The application token can be mount on your pods as volume. Every pods have a service tokens linked to the namespace where they are deployed. The deployments recreate the service accounts for each pods. 
Well that's all. Hope it's not confusing. Here is a schema of the principle of service account. I will appreciate all your comments. The commands on the schema are the most important for this topic. Thanks

![Service account.drawio.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1664235045070/f3Wfh5-X_.png align="left")
