---
layout: post
title: Rebuilding the EFI Directory in Centos7
---

----

### /boot exists but /boot/efi is lost or otherwise not functional

Due to data location conflicts, EFI partitions are often not set in an array on systems using software raid. However, the /boot partition has no such conflict, so it will more than likely have some sort of redundancy. Because of this, losing EFI while maintaining /boot's integrity is a not-unlikely scenario. The following presumes all /boot files are intact.

----


### Recreate the EFI partition if you need to, and mount it to /boot/efi

Use fdisk or gdisk or whatever you prefer to make an EFI partition if it doesn't already exist, 256MB is more than enough. Ensure the partition type is set as EFI so the correct flag is set. After the EFI partition is created, make the filesystem type fat32:

```shell
# /dev/sda3 is efi for this example
mkfs.fat -F32 /dev/sda3
```

Now you can mount the partition to /boot/efi:

```shell
# mkdir /boot/efi if it doesn't already exist, but it should
mount /dev/sda3 /boot/efi/
```

You'll also need to add a persistent mount entry in fstab for the system to come back after boot. Use blkid to show the UUID for the EFI partition and add to fstab as follows:

```shell
# blkid will output all device uuids
blkid

# add the entry to fstab, in this example uuid is F123-06D4
nano /etc/fstab
>>> UUID=F123-06D4 /boot/efi vfat defaults 0 2
```
----


### Rebuilding the EFI files

You can recreate most of the partition by simply running the following:

> This will require that networking is set up and active or that repo files are otherwise available to the system

```shell
yum reinstall grub2-efi grub2-efi-modules shim
```

Provided this completed without error, you should be able to look in your /boot/efi/ directory now and see that it has been populated with a bunch of new files, starting with /boot/efi/EFI/. These are generic files that form the framework for the EFI partition, but they still require an update to the grub.cfg file before they will work. If this is a CentOS system, there should be a /boot/efi/EFI/centos/ directory, and inside that is where we'll put the grub.cfg file. To create the following, simply run the following:

```shell
grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
```

> If using software raid and the grub2-mkconfig file outputs a 'can't find disk md0' or similar error, the problem is likely related to old raid info on the new array(s). The solution is to zero the superblock on the affected array and rebuild as necessary

You should now see a grub.cfg file in the specified directory. It will tell the system how to load partitions on boot, provided there were no errors output during the grub2-mkconfig command. If fstab has been updated with a mount entry for the EFI partition then you should be safe to exit out of the chroot environment and reboot the system.

On startup you should see the EFI OS entry in your boot list. If the EFI entry isn't present and you can't boot the system then you'll have to mount to a linux live session for further troubleshoot and recovery.
