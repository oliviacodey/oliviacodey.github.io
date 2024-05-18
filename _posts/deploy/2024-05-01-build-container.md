---
title: Build a Container
date: 2024-05-09
categories: [k8s,deploy]
tags: [k8s,deploy,container]     # TAG names should always be lowercase
---

## Podman

By default, Podman is configured with two container registries.

1.  [quay.io](https://quay.io/)
2.  [docker.io](https://hub.docker.com/)

You can find the default Podman container registry configuration in the following file.

```
/etc/containers/registries.conf
```

You can add custom or private container registries to this configuration. For example, google container registry, AWS ECR, self-hosted private registries, etc.

If you want to use other private container images from the registries, you can log in to the registry using `podman` command. For example, to login to the docker hub,

```
podman login docker.io
```

Once logged in, you will be able to pull the container images from the docker hub using `podman` command

If you wish to have a different registry configuration for a specific user, you can create separate `registries.conf` in the user directory with the container registry information.

```
$HOME/.config/containers/registries.conf
```

Each system user has its own container storage. Meaning, if you try to pull images from different user logins, it pulls the image from the remote registry instead of the local image.

For example,

1.  For **root** users, the containers are stored in `/var/lib/containers/storage` directory
2.  For other users, the containers are stored in `$HOME/.local/share/containers/storage/` directory

## Managing Containers With Podman

You can manage containers the same way you work with docker. However, instead of the docker command, we will use podman as a command with flags similar to docker. Also, you can run the podman command with any user without sudo privileges.

First, let’s try to pull an image. By default, podman searches for images in **quay.io** first and then in **docker.io**. If an image is not present in **quay.io**, podman searches in docker.io and pulls the image. So it is better to specify the full image name what the registry endpoint. For example,

```
podman pull docker.io/nginx
podman pull quay.io/quay/busybox
```

Let’s run an Nginx container from the dockerhub registry. The following command runs the Nginx container with `8080` host port mapping.

```
podman  run --name docker-nginx -p 8080:80 docker.io/nginx
```

You can check the mapped port using the following command. `-l` flat returns the details for the latest container.

```
podman port -l
```
