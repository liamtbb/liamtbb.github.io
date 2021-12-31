---
layout: post
title: Cloning Linux Systems with Rsync
---


----

### Removing cluster from Proxmox device without reinstalling:

So you set up a cluster in Proxmox and now you want to remove it. This can be done manually by running a few commands and removing a few files over the command line. This will stop the corosync service which will disable the Proxmox webgui, so perform steps via SSH or KVM to avoid disruption.

```shell
# stop the corosync and cluster service
systemctl stop pve-cluster corosync

# start cluster fileservice in normal mode
pmxcfs -l

# remove corosync files
rm /etc/corosync/*
rm /etc/pve/corosync.conf

# start filesystem services as normal
killall pmxcfs
systemctl start pve-cluster 
```

----

More detailed information can be found [here](https://pve.proxmox.com/wiki/Cluster_Manager)
