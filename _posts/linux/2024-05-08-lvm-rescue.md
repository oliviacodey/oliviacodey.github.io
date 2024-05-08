---
title: LVM in rescue mode
date: 2024-05-08
categories: [Linux,storage]
tags: [linux,lvm,rescue]     # TAG names should always be lowercase
---

## mount lvm in rescue mode

```bash
lvm vgscan -v
lvm_scan
/sysroot/sbin/vgchange -ay “volume group name”
lvm_scan
mount /dev/mapper/rhel.... /mnt/...
```
