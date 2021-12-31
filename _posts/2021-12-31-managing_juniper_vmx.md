---
layout: post
title: Managing Juniper VMX
---


----

### Managing vMX
This guide assumes vMX has already been configured. See [vMX installation instructions](https://www.juniper.net/documentation/us/en/software/vmx/vmx-getting-started/topics/concept/vmx-hw-sw-minimums.html) if host hasn't already been prepared, or if vmx.conf and vmx-junosdev.conf haven't already been set up.

----


### vMX via KVM

Management of vMX can be done using the vmx.sh script found in the vMX root directory. CD to the directory:
```shell
cd /vmx
```

First, the vMX instance will need initiating. Do so with:
```shell
sudo ./vmx.sh -lv --install
# If an error is presented, try running the command again. Errors will occasionally get thrown if the script tries to engage a service that hasn't fully started yet
```

After the vMX instance has been launched, you'll need to bind the network ports to the instance. This can be done with:
```shell
./vmx.sh --bind-dev
```

You can confirm the ports bound properly with:
```shell
./vmx.sh --bind-check
```

If the initialisation script ran without error and the bind-check confirmed the port configurations, the vMX instance should now be running. You can console into the control plane to make configuration changes or check on the startup status with:
```shell
# Substitute <vmx1> for name of instance if you're using non-standard naming, or have multiple running instances
./vmx.sh --console vcp vmx1
```

If you need to access the forwarding plane for troubleshooting, you can use:
```shell
# As with the control plane, substitute <vmx1> for the name of the intended instance
./vmx.sh --console vfp vmx1
```
----


### You can manage or determine status of the vMX instance with the following:

```shell
# start the instance:
./vmx.sh --start

# stop the instance:
./vmx.sh --stop

# restart the instance:
./vmx.sh --restart

# check instance status:
./vmx.sh --status
```
----


### Troubleshooting vMX port issues via Console

A bug with vMX seems to occur when the assigned license reaches its bandwidth limit. When this happens, one or more ports will get disabled to limit bandwidth, but the port will remain up and still learn ARP entries. This results in a system that is by all accounts functioning properly, but traffic simply dies as it enters the disabled port.

To see bandwidth usage across the PFE (packet forwarding engine), input the following in the vMX CLI:
```shell
show pfe statistics traffic bandwidth
```

If you suspect a port is bugging out, try pinging across an assigned PTP IP to confirm that the neighbour IP isn't responding. If you find that the neighbour IP isn't responding to ping attempts, but 'show route' is forwarding traffic to the correct port and 'show arp no-resolve' indicates that you're learning the neighbour IP on the port, then the bug is probably enacted and you need to reset the FPC...

To restart the FPC, input the following in the vMX CLI:
```shell
request chassis fpc slot 0 restart
```

Verify that the FPC is back online with the following:
```shell
show chassis fpc
```
