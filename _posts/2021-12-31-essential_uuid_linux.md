---
layout: post
title: Essential UUID Configuration in Linux
---


----

### Getting back to grub

Linux uses disk UUIDs to identify drives and partitions for mounting the filesystem. If you're having issues rebuilding grub, check that your UUID configurations correspond to the following guide.

----


### Essential UUID locations

There are 2 locations where UUIDs must be configured for a grub install to work: /etc/fstab and /etc/default/grub.

If you have software raid installed, you'll also have to confirm UUIDs in /etc/mdadm.conf.

----

### Determine device UUIDs

First, check your UUIDs using the **blkid** command. In this example we have disks /dev/sda and /dev/sdb; sda1+sdb1=md0 (/boot), sda2+sdb2=swap, sda3+sdb3=md1 (/):

```shell
# might need to run as sudo
sudo blkid

## output ##
/dev/md1: UUID="3bb093c7-5d42-4e5a-8a9c-56eb8c09fc28" TYPE="ext4"
/dev/sda1: UUID="a46bd833-5c08-4446-d8c8-870da85b7ba6" UUID_SUB="821b47e6-575a-2e21-e8e8-5e771ba47f30" LABEL="test-srv-1:0" TYPE="linux_raid_member" PARTUUID="0006650c-01"
/dev/sda2: UUID="f7407241-6b5b-4cd9-94a2-1a80252617f7" TYPE="swap" PARTUUID="0006650c-02"
/dev/sda3: UUID="d64a28c9-759f-b90f-35af-5a05805a92d9" UUID_SUB="3f87ba6d-b831-d09e-5ae7-623132e803b8" LABEL="test-srv-1:1" TYPE="linux_raid_member" PARTUUID="0006650c-03"
/dev/sdb1: UUID="a46bd833-5c08-4446-d8c8-870da85b7ba6" UUID_SUB="d45afe3b-3844-05a7-6093-0a7904696137" LABEL="test-srv-1:0" TYPE="linux_raid_member" PARTUUID="0007e0c3-01"
/dev/sdb2: UUID="6989461f-3f81-4480-b0ed-cb395a0c5979" TYPE="swap" PARTUUID="0007e0c3-02"
/dev/sdb3: UUID="d64a28c9-759f-b90f-35af-5a05805a92d9" UUID_SUB="445f2f00-7ae6-2df0-ce42-f2312b410139" LABEL="test-srv-1:1" TYPE="linux_raid_member" PARTUUID="0007e0c3-03"
/dev/md0: UUID="4404b70b-ebd4-4320-b77f-15e4e16da773" TYPE="ext4"
```

Note that the UUIDs are identical for partitions in the same array, but sub-UUIDs are different.

----


### /etc/fstab

Now let's take a look at the fstab file:

```shell
cat /etc/fstab

## output ##
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/md1 during installation
UUID=3bb093c7-5d42-4e5a-8a9c-56eb8c09fc28 /               ext4    errors=remount-ro 0       1
# /boot was on /dev/md0 during installation
UUID=4404b70b-ebd4-4320-b77f-15e4e16da773 /boot           ext4    defaults        0       2
# swap was on /dev/sda2 during installation
UUID=f7407241-6b5b-4cd9-94a2-1a80252617f7 none            swap    sw              0       0
# swap was on /dev/sdb2 during installation
UUID=6989461f-3f81-4480-b0ed-cb395a0c5979 none            swap    sw              0       0
```

Note the first 2 entries for / and /boot. 

**/** is using the UUID for device **/dev/md1**.

**/boot** is using the UUID for device **/dev/md0**.

Fstab uses the UUID for the array when creating entries.

----


### /etc/default/grub

When the grub.cfg file is generated, it uses data from both the fstab and /etc/default/grub files. Some distros will have UUIDs in the grub file that also need validating.

Run the following to check the file for UUIDs:

```shell
cat /etc/default/grub

## output ##
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.md.uuid=d64a28c9:759fb90f:35af5a05:805a92d9 rd.md.uuid=a46bd833:5c084446:d8c8870d:a85b7ba6 rhgb quiet net.ifnames=0 biosdevname=0"
GRUB_DISABLE_RECOVERY="true"
```

In this case we have UUIDs in grub. Note that there are 2 entries, the first is / and the second is /boot.

**/** is using the UUID for devices **sda3/sdb3**.

**/boot** is using the UUID for devices **sda1/sdb1**.

grub uses the UUID for the partition when creating entries.

----


### mdadm.conf

/etc/mdadm.conf (or /etc/mdadm/mdadm.conf) is where mdadm stores array configuration data for assembly during bootup. If you're using mdadm, this is approximately what you should see:

```shell
cat /etc/mdadm.conf

## output ##
# mdadm.conf
#
# Please refer to mdadm.conf(5) for information about this file.
#

# by default (built-in), scan all partitions (/proc/partitions) and all
# containers for MD superblocks. alternatively, specify devices to scan, using
# wildcards if desired.
#DEVICE partitions containers

# auto-create devices with Debian standard permissions
CREATE owner=root group=disk mode=0660 auto=yes

# automatically tag new arrays as belonging to the local system
HOMEHOST <system>

# instruct the monitoring daemon where to send mail alerts
MAILADDR monitors@fakedomain.com

# definitions of existing MD arrays
ARRAY /dev/md/1 metadata=1.2 UUID=d64a28c9:759fb90f:35af5a05:805a92d9 name=test-srv-1:1
ARRAY /dev/md/0 metadata=1.2 UUID=a46bd833:5c084446:d8c8870d:a85b7ba6 name=test-srv-1:0

# This file was auto-generated on Wed, 28 Jan 2015 17:42:19 -0800
# by mkconf 3.2.5-5
```

Order of the arrays doesn't matter here, we just need all md# to be accounted for. Here we can see md1 (/) and md0 (/boot).

**/** array md1 is using the UUID for devices **sda3/sdb3**.

**/boot** array md0 is using the UUID for devices **sda1/sdb1**.

grub uses the UUID for the partition when creating entries.

----


### Entries confirmed?

That's more or less where and how all the essential UUIDs are configured. Update your entries to correspond then run grub-update or grub2-mkconfig (or whatever your distro uses to generate the grub.cfg file) and you should now have a bootable system.
