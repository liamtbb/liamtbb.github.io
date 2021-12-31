---
layout: post
title: Mounting an Unbootable Linux System
---


----
### Preparing the System for Recovery

- To start, create a directory for mounting the system root:

```shell
mkdir /mnt/sysimage
```

- Now find the root directory you want to mount. If the root was part of a RAID1 or other such array, you'll have to recreate the array before you can mount it. Enter the following to view the available drives and partitions, including their filesystem types:

```shell
lsblk -f
```

- Hopefully you can determine which partition is which based on context clues or prior knowledge of the system. In this case we have /dev/sdb1 (shows as a 512MB linux raid partition) and /dev/sdb3 (shows as a 950GB linux raid partition) and /dev/sdb2 (shows as 8GB partition, probably SWAP. We'll need to bring the array partitions online before we can mount them, and we'll need to force mdadm to start the array with just a single partition for each. Do so with the following:

```shell
$ mdadm -A -R /dev/md0 /dev/sdb1
$ mdadm -A -R /dev/md1 /dev/sdb3
```

- Another 'lsblk' should now show /dev/sdb1 and /dev/sdb3 as part of their respective MD arrays, as well as what type of RAID they're configured as. We now have to determine if we want to mount the raid arrays directly, or if the arrays are just containers for logical volumes which will need to be mounted individually. We can do this with the 'vgscan' command, which will search for available volume groups:

```shell
vgscan
```
----


### Mounting with Logical Volumes

- In this case, 'vgscan' confirms that vggroup 'dv_scll' exists, and that it contains logical volumes LV_ROOT, LV_HOME, LV_USR, LV_TMP, and LV_VAR. Fortunately these LVs are labeled in a logical way the eliminates the guesswork of what they do. If that wasn't the case, you'd have to mount each LV individually and determine what it was based on the files within. Since we already know where they belong, we can mount them accordingly:

!!Note that '/' is mounted to /mnt/sysimage on the recovery system, so all other partitions must be mounted in a cascading fashion using /mnt/sysimage/ as the root for the recovered system. This should make more sense when applied to the example below:!!

```shell
mount /dev/dv_scll/LV_ROOT /mnt/sysimage/
mount /dev/dv_scll/LV_HOME /mnt/sysimage/home
mount /dev/dv_scll/LV_USR /mnt/sysimage/usr
mount /dev/dv_scll/LV_TMP /mnt/sysimage/tmp
mount /dev/dv_scll/LV_VAR /mnt/sysimage/var
```
- If there are mounting errors, it's possible the partition/LV needs a filesystem check. Run an 'fsck' or similar on the problem device.

----


### Mounting with Physical Volumes or Arrays

- If 'vgscan' shows no available logical volumes, or you already know the system uses a certain partition scheme, you can mount the physical volumes directly. In this case we know that /dev/md0 is '/boot', /dev/md1 is '/', and /dev/sdb2 is SWAP (which isn't necessary right now and can be ignored). Mounting is done the same as with the LVs above:

```shell
mount /dev/md1 /mnt/sysimage/
mount /dev/md0 /mnt/sysimage/boot
```

- If you have more partitions you can add them as needed, the example used is a basic setup common to many of our installs with a small partition for boot and most of the remaining space going to root with a little bit left over for SWAP.

----


### Recovered System Final Steps

- Provided that the partitions all mounted without issue, we still need to tell the system to bind a few other important mount points to the //recovered// system instead of the //recovery// system:

```shell
mount --bind /proc /mnt/sysimage/proc
mount --bind /sys /mnt/sysimage/sys
mount --bind /dev /mnt/sysimage/dev
mount --bind /dev/pts /mnt/sysimage/dev/pts
```

- If there were no errors, we should now be able to 'chroot' to the recovered system's mount directory:

```shell
chroot /mnt/sysimage
```

- We can now use the system as if we booted right into it. You can turn up the network if need be, install packages, install grub if the system was missing it, turn up services, etc. Exiting out of the shell will return us to the shell of the recovery system.
