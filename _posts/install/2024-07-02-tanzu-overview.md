---
title: Tanzu Overview
date: 2024-07-02
categories: [k8s,Tazu,Overview]
tags: [k8s,tanzu]     # TAG names should always be lowercase
---

## Tanzu

Tanzu is for vSphere admins to delegate resources to Kubernetes kuster admins so they can delegate kubernetes clusters to the delelopers.

## Networking

* Management Network
* Workload Network
* Frontend Network

The Supervisor Cluster needs Workload Management enabled.

Workload management needs vSphere DRS and vSphere HA

The networks needs to be isolated on L2
Supervisor cluster has one nic in the management network and one nic in the workload cluster.
The workload cluster needs to access the frontend network.

## Install

```bash
curl -sfL https://get.k3s.io | sh -
```
