---
layout: post
title: Cloning Linux Systems with Rsync
---


----

## Launch a live linux session on a blank destination system, partition your drive(s), format and mount partitions, establish networking.
You can create the partitions from the live session, or you can install a new bootable system using the same base OS as the source server prior to mounting. If you install a new bootable system then you have the option of preserving some essential files rather than rebuilding after clone (mdadm.conf, fstab, grub, etc.)

We won't go in to partitioning here as that is a post unto itself and will already be covered more effectively elsewhere. However, when creating partitions for the clone, ensure that new partitions have adequate capacity to match or exceed current usage on the source system.

The live linux session will need to have networking configured with a private or public IP that can ping/ssh to the source system.

I typically create a directory for mounting the root (/) partition to first, then I mount all the other partitions on top. Let's say you have partitions md0 (/boot), md1 (/), and md2 (/home):

```shell
#create mount directory for destination system
$ mkdir /mount

#mount destination partitions
$ mount /dev/md1 /mount/   ## mount root
$ mount /dev/md0 /mount/boot/   ## mount boot
$ mount /dev/md2 /mount/home/   ## mount home
```
----

## Rsync directories from source to destination server.
Replace 1.2.3.4 with the IP of your source system. This is ideally performed while the source system is offline and mounted to a live linux session, or with primary services deactivated to minimise data change during sync. By default, rsync only updates file differentials, so you can always run rsync afterwards to quickly sync any deltas between the source and destination devices.

The below rsync command will sync everything (a) including hard-links (H), while excluding anything with the '--exclude=' flag. If you're cloning overtop of a bootable system then you should also exclude the grub/default, etc/fstab, and etc/mdadm.conf files so you don't have to rebuild them (or at least make a .bak copy of each for recovery after). Remove the '--exclude=/etc/mdadm.conf' if you aren't using software raid.

```
# if you're cloning over a totally clean, unbootable system
$ rsync -aHxv --numeric-ids --progress root@1.2.3.4:/* /mount --exclude=/dev --exclude=/proc --exclude=/sys --exclude=/tmp

# if you're cloning over a working, bootable system
$ rsync -aHxv --numeric-ids --progress root@1.2.3.4:/* /mount --exclude=/dev --exclude=/proc --exclude=/sys --exclude=/tmp --exclude=/etc/default/grub --exclude=/etc/fstab --exclude=/etc/mdadm.conf
```
----

## Update your fstab, default/grub and mdadm.conf files as needed:

```
# ls or blkid to retrieve new uuids
$ ls -la /dev/disk/by-uuid # to get new UUID's 
$ blkid

# update essential boot files
$ nano /mount/etc/fstab
$ nano /mount/etc/default/grub
$ nano /mount/etc/mdadm.conf
```
----

## Mount /mount/proc, /mount/sys/, /mount/dev/, chroot to /mount:
These directories need to be mounted for chroot to work. Chroot simply shifts the functional filesystem root to the chosen directory, in this case our mounted clone server. Chrooting will allow us to rebuild grub so the cloned system will boot on its own.

```
$ cd /mount/ 
$ mount -t proc proc proc/ 
$ mount -t sysfs sys sys/ 
$ mount -o bind /dev dev/ 
$ chroot .
```
----

## If you get errors using commands after chrooting then you might have to export /bin to PATH:
May or may not be an issue. If it isn't then great, if it is then slap the following commands in and it should be back to great again.

```
$ export PATH=$PATH:/bin
$ export PATH=$PATH:/usr/sbin
```
----

## Install grub or other bootloader of your choosing
This is where the updated fstab/grub/mdadm files come into play. Grub will use the new system's UUIDs to organise the partitions on boot, so all UUIDs will need to be updated accordingly for this to work. If you installed over a working, bootable system and preserved the old files then you should be ready to go. The process might be different depending on the distro being cloned, and the grub.cfg file might not be in a slightly different directory in /boot/grub. Here are examples for Centos7 and Ubuntu18:

```
#update grub.cfg file with new UUIDs
$ grub2-mkconfig -o /boot/grub/grub.cfg   ## Centos7
OR
$ update-grub   ## Ubuntu18

#install grub to bootable drives
$ grub2-install /dev/sda
$ grub2-install /dev/sdb   ## install to multiple drives if available
```
----

## Cross fingers and reboot
If you didn't get any errors after running the above grub commands then the system is, at least in theory, ready to boot. You can exit out of chroot and reboot the device. On boot, select the appropriate system drive to confirm everything is operational.

If boot fails, mount the system back up for troubleshooting. Chances are there was an issue with the configured UUIDs in the grub/fstab/mdadm files. Compare them against the source system to help identify problems.

----
