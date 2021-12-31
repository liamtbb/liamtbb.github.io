---
layout: post
title: Cloning Linux Systems with Rsync
---




- Install and fully configure the first system, start ssh server.

- Launch a live linux session on the blank system, partition your drive, format partitions and mount them to /mount, /mount/boot, /mount/home/, etc...

- Run (replace the IP with your configured and installed source system):
```
$ rsync -aHxv --numeric-ids --progress root@1.2.3.4:/* /mount --exclude=/dev --exclude=/proc --exclude=/sys --exclude=/tmp
```

- Update your fstab, default/grub and mdadm.conf files as needed:
```
$ ls -la /dev/disk/by-uuid # to get new UUID's 
$ nano /mount/etc/fstab
$ nano /mount/etc/default/grub
$ nano /mount/etc/mdadm.conf
```

- Mount /mount/proc, /mount/sys/, /mount/dev/, chroot to /mount:
```
$ cd /mount/ 
$ mount -t proc proc proc/ 
$ mount -t sysfs sys sys/ 
$ mount -o bind /dev dev/ 
$ chroot .
```

- If you get errors using commands after chrooting then you might have to export /bin to PATH:
```
$ export PATH=$PATH:/bin
$ export PATH=$PATH:/usr/sbin
```

- Install grub or other bootloader of your choosing
```
#update grub.cfg file with new UUIDs
$ grub2-mkconfig -o /boot/grub/grub.cfg

#install grub to bootable drives
$ grub2-install /dev/sdb
```

- Cross fingers and reboot
