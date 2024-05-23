---
title: InitContainers
date: 2024-05-22
categories: [k8s,deploy]
tags: [k8s,initcontainers,deployment]     # TAG names should always be lowercase
---

## Understanding init containers

A Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.

Init containers are exactly like regular containers, except:

* Init containers always run to completion.
* Each init container must complete successfully before the next one starts.

If you specify multiple init containers for a Pod, kubelet runs each init container sequentially. Each init container must succeed before the next can run.

* Init containers can contain utilities or custom code for setup that are not present in an app image. For example, there is no need to make an image `FROM` another image just to use a tool like `sed`, `awk`, `python`, or `dig` during setup.
* The application image builder and deployer roles can work independently without the need to jointly build a single app image.
* Init containers can run with a different view of the filesystem than app containers in the same Pod. Consequently, they can be given access to [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) that app containers cannot access.
* Because init containers run to completion before any app containers start, init containers offer a mechanism to block or delay app container startup until a set of preconditions are met. Once preconditions are met, all of the app containers in a Pod can start in parallel.
* Init containers can securely run utilities or custom code that would otherwise make an app container image less secure. By keeping unnecessary tools separate you can limit the attack surface of your app container image.

### Examples

Here are some ideas for how to use init containers:

* Wait for some time before starting the app container with a command like

  ```bash
  sleep 60
  ```

* Clone a Git repository into a [Volume](https://kubernetes.io/docs/concepts/storage/volumes/)

* Place values into a configuration file and run a template tool to dynamically generate a configuration file for the main app container. For example, place the `POD_IP` value in a configuration and generate the main app configuration file using Jinja.

### Init containers in use

This example defines a simple Pod that has two init containers. The first waits for `myservice`, and the second waits for `mydb`. Once both init containers complete, the Pod runs the app container from its `spec` section.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

kubectl get -f myapp.yaml

```
NAME        READY   STATUS     RESTARTS   AGE
myapp-pod   0/1     Init:0/2   0          19s
```

You can see that the pod is unable to start due to the initcontainer.

Make the pod be able to start by running the following

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

kubectl get -f myapp.yaml
```
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   1/1     Running   0          2m52s
```