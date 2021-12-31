---
layout: post
title: Removing Cluster from Proxmox Device
---


----

### Removing cluster from Proxmox device without reinstalling:

So you set up a cluster in Proxmox and now you want to remove it. This can be done manually by running a few commands and removing a few files over the command line. This will stop the corosync service which will disable the Proxmox webgui, so perform steps via SSH or KVM to avoid disruption.

Stop the corosync and cluster service:
```shell
systemctl stop pve-cluster corosync
```

Start cluster fileservice in normal mode:
```shell
pmxcfs -l
```

Remove corosync files:
```shell
rm /etc/corosync/*
rm /etc/pve/corosync.conf
```

Start filesystem services as normal:
```shell
killall pmxcfs
systemctl start pve-cluster 
```
----

More detailed information can be found [here](https://pve.proxmox.com/wiki/Cluster_Manager)
