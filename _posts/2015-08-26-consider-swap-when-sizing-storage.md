---
title: Remember dump and swap when sizing storage
slug: consider-swap-when-sizing-storage
date_published: 2015-08-26T16:22:08.173Z
date_updated:   2015-08-26T16:23:15.445Z
---

Remember to reserve space for swap and for dump when designing the storage for a system. In general, you'll need to reserve space on the primary "zones" volume equal to 150% of the amount of physical RAM installed. 

Insufficient space on the "zones" pool for a dump volume or a swap volume will cause initial configuration to fail without warning, until you reboot, land in maintenance mode, and find a message like this:

```
svc:/system/filesystem/smartdc:default: Method "/lib/svc/method/fs-joyent" failed with exit status 95.
```

Swap
----
As a rule of thumb in any system, the swap device should have at least as much capacity as the installed DRAM. In SmartOS, this is more of a requirement, according to [comments in the initial configuration script](https://github.com/joyent/smartos-live/blob/master/overlay/generic/smartdc/lib/smartos_prompt_config.sh#L388-L395). 

That script will attempt to allocate a zvol for swap equal to the size of installed DRAM during initial configuration. For a system with lots of RAM and a small system "zones" pool, this may be impractical (because you don't have enough space left over for zones), or impossible (because you don't even have enough space for swap).

If it is in fact impossible, the swap volume will not be created at all, and you will need to create it manually from noinstall mode:

```
zpool import zones
zfs create -V {Installed RAM}G zones/swap
swap -a /dev/zvol/dsk/zones/swap
zpool export zones
reboot
```

There is really only one approach to take here, and that is to make sure that you have at least as much space on your pool as you have RAM.

Dump
----
Again as a rule of thumb, the dump device should be large enough to store the contents of memory in use by the kernel at the time of a system crash. [Classical Solaris documentation](http://docs.oracle.com/cd/E23824_01/html/E24456/device-7.html) puts this at 50% to 75% of installed RAM. 

[The SmartOS configuration script](https://github.com/joyent/smartos-overlay/blob/release-20130418/smartdc/bin/smartos_prompt_config.sh#L299-L314) takes a different approach, however, allocating a dump zvol of 5% of total pool size or 4GB, whichever is smaller. On systems with lots of RAM, this may be too small. On startup, if `dumpadm` cannot allocate the device, it will fail, causing the error above.

The solution is to reboot into noinstall mode, import the zones pool manually, and then set the dump zvol volsize to half of your installed RAM.

```
zpool import zones
zfs set volsize={Installed RAM ÷ 2}G zones/dump
dumpadm -d /dev/zvol/dsk/zones/dump 
zpool export zones
reboot
```

Of course, this means half your RAM's worth of your zpool goes to dump, in addition to the space required for swap. 

Simple worksheet
------------
How much of your zpool do you reserve for dump and swap?

Installed RAM × 1.5.

For example, if your system has 256GB RAM and a (nominally) 1TB SSD zpool, (256×1.5)=384GB of your pool goes to swap and dump, leaving 616GB for your zones. Fudge for overhead, and you're down to more like 570GB usable space for zones.

