---
title: Resize LVM partition
date: 2024-05-08
categories: [Linux,storage]
tags: [linux,lvm,resize]     # TAG names should always be lowercase
---

### Extend the size in vmware

## Make sure linux detects the new physical size
```bash
lsblk
echo 1 > /sys/block/sda/device/rescan
lsblk
```

## Extend the partition

```bash
parted
print free
resizepart 2 100%F
quit
```

## Resize the LVM

```bash
pvresize /dev/sda2
vgs
lvextend /dev/mapper/lvm-partition -l +100%FREE -r # (+L 20g)
df -h
```
