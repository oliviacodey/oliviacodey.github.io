---
title: Kubernetes
date: 2024-05-06
categories: [Kubernetes, Install]
tags: [k8s, argocd, metallb]     # TAG names should always be lowercase
---

# Kubernetes

## Get started

### k3s

```bash
curl -sfL https://get.k3s.io | sh -
```

### Om kubectl behövs installeras

skapa repo-fil

```yaml
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
```

installera kubectl och anslut till klustret

```bash
dnf install kubectl -y
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
kubectl get pods
```

aktivera auto-complete

```bash
dnf install bash-completion
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
```

## MetalLB

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
  labels:
    app: metallb
```

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
```

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.148.20 - 192.168.148.25
```

## Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

## NFS StorageClass

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=x.x.x.x \
    --set nfs.path=/exported/path
```

## argocd

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

```bash
kubectl -n argocd exec -it pod/argocd-server-7cbbdb87d7-tj85z -- argocd admin initial-password
```

<http://argocd-ip>
admin:initial-password

kubectl config get-contexts -o name

## ConfigMaps

### Skapa en configmap från fil

kubectl create configmap mediawiki --from-file /home/micke/LocalSettings.php

### Mounta en configmap i en pod

```yaml
  volumeMounts:
    - name: config-volume
      mountPath: /var/www/html/LocalSettings.php
      subPath: LocalSettings.php #värdet i configmapen
volumes:
- name: config-volume
  configMap:
    name: mediawiki
```

### Restart deployment

```bash
kubectl rollout restart deployment/nginx
```

## kubectl

* kubectl -w (wait)
* kubectl port-forward service serviceName 8443:443
* kubectl get pods -o=wide -n=<namespace_name>
* kubectl logs -l name="label name"
* kubectl config get-contexts
* kubectl config use-context "cluster name"
* kubectl logs --since=6h
* kubectl exec -it <pod_name> /bin/sh

## apply

Apply a configuration change to a resource from a file or stdin

```bash
kubectl apply -f FILENAME [flags]
```

### Change default namespace

```bash
kubectl config set-context --current --namespace=argocd
```

## annotate
* Add or update the annotations of one or more resources
- kubectl annotate (-f FILENAME | TYPE NAME | TYPE/NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--overwrite] [--all] [--resource-version=version] [flags]
You can use either labels or annotations to attach metadata to Kubernetes objects. Labels can be used to select objects and to find collections of objects that satisfy certain conditions. In contrast, annotations are not used to identify and select objects. The metadata in an annotation can be small or large, structured or unstructured, and can include characters not permitted by labels. It is possible to use labels as well as annotations in the metadata of the same object.
- Example: Build, release, or image information like timestamps, release IDs, git branch, PR numbers, image hashes, and registry address

```yaml
"metadata": {
  "annotations": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

cordon/uncordon	kubectl cordon NODE [options]	Mark node as unschedulable.
cp	kubectl cp <file-spec-src> <file-spec-dest> [options]	Copy files and directories to and from containers.
create	kubectl create -f FILENAME [flags]	Create one or more resources from a file or stdin.
delete	kubectl delete (-f FILENAME | TYPE [NAME | /NAME | -l label | --all]) [flags]	Delete resources either from a file, stdin, or specifying label selectors, names, resource selectors, or resources.
describe	kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | /NAME | -l label]) [flags]	Display the detailed state of one or more resources.
diff	kubectl diff -f FILENAME [flags]	Diff file or stdin against live configuration.
drain	kubectl drain NODE [options]	Drain node in preparation for maintenance.
edit	kubectl edit (-f FILENAME | TYPE NAME | TYPE/NAME) [flags]	Edit and update the definition of one or more resources on the server by using the default editor.
events	kubectl events	List events
exec	kubectl exec POD [-c CONTAINER] [-i] [-t] [flags] [-- COMMAND [args...]]	Execute a command against a container in a pod.
explain	kubectl explain TYPE [--recursive=false] [flags]	Get documentation of various resources. For instance pods, nodes, services, etc.
expose	kubectl expose (-f FILENAME | TYPE NAME | TYPE/NAME) [--port=port] [--protocol=TCP|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type] [flags]	Expose a replication controller, service, or pod as a new Kubernetes service.
get	kubectl get (-f FILENAME | TYPE [NAME | /NAME | -l label]) [--watch] [--sort-by=FIELD] [[-o | --output]=OUTPUT_FORMAT] [flags]	List one or more resources.
logs	kubectl logs POD [-c CONTAINER] [--follow] [flags]	Print the logs for a container in a pod.
options	kubectl options	List of global command-line options, which apply to all commands.
patch	kubectl patch (-f FILENAME | TYPE NAME | TYPE/NAME) --patch PATCH [flags]	Update one or more fields of a resource by using the strategic merge patch process.
plugin	kubectl plugin [flags] [options]	Provides utilities for interacting with plugins.
port-forward	kubectl port-forward POD [LOCAL_PORT:]REMOTE_PORT [...[LOCAL_PORT_N:]REMOTE_PORT_N] [flags]	Forward one or more local ports to a pod.
rollout	kubectl rollout SUBCOMMAND [options]	Manage the rollout of a resource. Valid resource types include: deployments, daemonsets and statefulsets.
  ### Rollback to the previous deployment
  kubectl rollout undo deployment/abc
  ### Check the rollout status of a daemonset
  kubectl rollout status daemonset/foo
  ### Restart a deployment
  kubectl rollout restart deployment/abc
  ### Restart deployments with the 'app=nginx' label
  kubectl rollout restart deployment --selector=app=nginx
run	kubectl run NAME --image=image [--env="key=value"] [--port=port] [--dry-run=server|client|none] [--overrides=inline-json] [flags]	Run a specified image on the cluster.
top	kubectl top (POD | NODE) [flags] [options]	Display Resource (CPU/Memory/Storage) usage of pod or node.
version	kubectl version [--client] [flags]	Display the Kubernetes version running on the client and server.



api-resources	kubectl api-resources [flags]	List the API resources that are available.
api-versions	kubectl api-versions [flags]	List the API versions that are available.
auth	kubectl auth [flags] [options]	Inspect authorization
certificate	kubectl certificate SUBCOMMAND [options]	Modify certificate resources.
cluster-info	kubectl cluster-info [flags]	Display endpoint information about the master and services in the cluster.
config	kubectl config SUBCOMMAND [flags]	Modifies kubeconfig files. See the individual subcommands for details.
convert	kubectl convert -f FILENAME [options]	Convert config files between different API versions. Both YAML and JSON formats are accepted. Note - requires kubectl-convert plugin to be installed.
proxy	kubectl proxy [--port=PORT] [--www=static-dir] [--www-prefix=prefix] [--api-prefix=prefix] [flags]	Run a proxy to the Kubernetes API server.
replace	kubectl replace -f FILENAME	Replace a resource from a file or stdin.
scale	kubectl scale (-f FILENAME | TYPE NAME | TYPE/NAME) --replicas=COUNT [--resource-version=version] [--current-replicas=count] [flags]	Update the size of the specified replication controller.
scale	kubectl scale (-f FILENAME | TYPE NAME | TYPE/NAME) --replicas=COUNT [--resource-version=version] [--current-replicas=count] [flags]	Update the size of the specified replication controller.
set	kubectl set SUBCOMMAND [options]	Configure application resources.
taint	kubectl taint NODE NAME KEY_1=VAL_1:TAINT_EFFECT_1 ... KEY_N=VAL_N:TAINT_EFFECT_N [options]	Update the taints on one or more nodes.
- Node affinity is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite -- they allow a node to repel a set of pods.
wait	kubectl wait ([-f FILENAME] | resource.group/resource.name | resource.group [(-l label | --all)]) [--for=delete|--for condition=available] [options]	Experimental: Wait for a specific condition on one or many resources.
