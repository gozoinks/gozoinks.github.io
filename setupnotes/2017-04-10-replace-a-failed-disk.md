---
title: Replace a failed disk
slug: replace-a-failed-disk
date_published: 2017-04-10T16:55:17.293Z
date_updated:   2017-04-11T20:36:38.279Z
---

SmartOS automatically disconnects a disk that has thrown too many errors, but you have to replace it yourself.

You'll need to manually identify the failed disk and its interface, remove the failed disk, attach the new disk, and tell the pool to use the new disk. 

This procedure assumes that you have hot-swap drive bays and can service the equipment without powering down or rebooting.

If you have a hot spare, you probably just need to do a `zpool replace <pool> <failed device> <spare device>`, and this procedure does not strictly apply.


# Quick summary
1. Find the failed device name: 
`zpool status <pool>` 
2. Find the attachment point of the failed device: 
`cfgadm -l`
3. Stop the failed device: 
`zpool offline <pool> <device>`
4. Swap disks
5. Attach the new device: 
`cfgadm -f -c <attachment point from step 2>`
6. Start resilvering: 
`zpool replace <pool> <device>`

#Details

# 1. Find the name of the failed device

## `zpool status <pool>`

The disk marked 'FAULTED' is the one to replace.

```
[root@node5 ~]# zpool status zones
  pool: zones
 state: DEGRADED
status: One or more devices are faulted in response to persistent errors.
        Sufficient replicas exist for the pool to continue functioning in a
        degraded state.
action: Replace the faulted device, or use 'zpool clear' to mark the device
        repaired.
  scan: resilvered 238G in 3h24m with 0 errors on Thu Apr  6 20:04:42 2017
config:

        NAME        STATE     READ WRITE CKSUM
        zones       DEGRADED     0     0     0
          raidz2-0  DEGRADED     0     0     0
            c1t0d0  ONLINE       0     0     0
            c1t1d0  ONLINE       0     0     0
            c1t2d0  ONLINE       0     0     0
            c1t3d0  ONLINE       0     0     0
            c1t4d0  FAULTED      1     6     0  too many errors
            c1t5d0  ONLINE       0     0     0

```

The pool in this example is degraded but otherwise functioning, so it should be safe to continue. `c1t4d0` is the disk to replace. 

Before moving on, however, you might want to see what the system can tell you about the nature of the failure.

## `iostat -en`

This command shows a more concise summary of disk health, along with specifics about the nature of the errors.

In this example, c1t4d0 has three soft errors, one hard error, and eleven transport errors:

```
[root@node5 ~]# iostat -en
  ---- errors --- 
  s/w h/w trn tot device
    0   0   0   0 lofi1
    0   0   0   0 ramdisk1
    0   0   0   0 c0t0d0
    0   0   0   0 c1t0d0
    0   0   0   0 c1t1d0
    0   0   0   0 c1t2d0
    0   0   0   0 c1t3d0
    3   1  11  15 c1t4d0
    0   0   0   0 c1t5d0
    0   0   0   0 zones
```

Hard errors and transport errors generally indicate a hardware malfunction. Errors on more than one disk might indicate a problem with the controller or with the backplane. In this case, only one disk is showing such errors, so the malfunction is most likely associated with that disk only. 

In order to choose the right replacement disk, it might help to get the configuration and model number of the failed disk.

### `iostat -En <device>`

This invocation (with a capital 'E') provides more details, including the model and capacity of the affected disk. In this example, the failed disk is a 2-terabyte Seagate ST2000NM0011.

```
[root@node5 ~]# iostat -En c1t4d0
c1t4d0           Soft Errors: 3 Hard Errors: 1 Transport Errors: 11 
Vendor: ATA      Product: ST2000NM0011     Revision: SN03 Serial No: Z1P3DF18 
Size: 2000.40GB <2000398934016 bytes>
Media Error: 0 Device Not Ready: 0 No Device: 0 Recoverable: 0 
Illegal Request: 71 Predictive Failure Analysis: 0 
```

Having identified the device in the pool that is faulted, the nature of the fault, and some details about the device, the next thing you need to know is the interface where you will disconnect the old drive and reattach the new one.

# 2. Get the attachment point of the failed device


### `cfgadm -l`

This tool lists storage interfaces, the devices attached to them, and the status of those devices. In this example, the sata0/4 interface—where we would expect to see our failed c1t4d0 device—shows a disconnected receptacle, an unconfigured occupant, and a general 'failed' condition:

```
[root@node5 ~]# cfgadm -l
Ap_Id                          Type         Receptacle   Occupant     Condition
sata0/0::dsk/c1t0d0            disk         connected    configured   ok
sata0/1::dsk/c1t1d0            disk         connected    configured   ok
sata0/2::dsk/c1t2d0            disk         connected    configured   ok
sata0/3::dsk/c1t3d0            disk         connected    configured   ok
sata0/4                        sata-port    disconnected unconfigured failed
sata0/5::dsk/c1t5d0            disk         connected    configured   ok
usb0/1                         usb-storage  connected    configured ok
usb0/2                         unknown      empty        unconfigured ok
usb1/1                         unknown      empty        unconfigured ok
usb1/2                         unknown      empty        unconfigured ok
```

So now you know that the failed disk was connected to `sata0/4`. That is the attachment point name you will need in order to disconnect it and reattach it.

# 3. Stop the old disk

A meticulous administrator avoids surprising the machine. Here is the polite way to advise the system that you are about to yank out the failed disk:

## `zpool offline zones <device>`

The failed disk is most likely already offline, but this command gives the system a chance to warn you if proceeding might cause bad things to happen—like if you have identified the wrong device, or if taking this device offline will remove too many members for the pool to continue operating. The command should complete without error.

```
[root@node5 ~]# zpool offline zones c1t4d0
bringing device c1t4d0 offline
```

Now you're clear to take out the disk and install the new one.

# 4. Physically replace the disks.
Use your eyes and hands to remove the failed disk and install the replacement.

# 5. Attach the new disk

## `cfgadm -f -c connect <attachment point>`

With the new disk in place, use `cfgadm` to attach it to the storage interface (-f for force, -c for 'change state'):

```
[root@node5 ~]# cfgadm -f -c connect sata0/4
Activate the port: /devices/pci@0,0/pci15d9,f280@1f,2:4
This operation will enable activity on the SATA port
Continue (yes/no)? yes
```

It will tell you the hardware device path of the storage interface and ask you to confirm. `cfgadm -l` should list the newly attached device as connected, configured and ok.

If it does not automatically appear configured, you will need to help it along:

## `cfgadm -f -c configure <attachment point>`

If this completes without error, `cfgadm -l` should now show the device as configured.

Now it's time to tell ZFS to bring it back into the pool.

# 6. Start resilvering

## `zpool replace <pool> <device>`

When you're ready to start the rebuild, tell the pool to replace the old disk with the disk you've just installed:

```
[root@node5 ~]# zpool replace zones c1t4d0
```

If the command exits with an error, then you may be working with a disk that has been previously formatted, or its geometry might be incompatible with the other disks in your pool, or any of a number of esoteric conditions may be interfering.

Use `zpool status` to see the results of the command:

```
[root@node5 ~]# zpool status
  pool: zones
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Mon Apr 10 14:37:13 2017
    46.0M scanned out of 7.71T at 5.75M/s, 390h53m to go
    7.94M resilvered, 0.00% done
config:

        NAME              STATE     READ WRITE CKSUM
        zones             DEGRADED     0     0     0
          raidz2-0        DEGRADED     0     0     0
            c1t0d0        ONLINE       0     0     0
            c1t1d0        ONLINE       0     0     0
            c1t2d0        ONLINE       0     0     0
            c1t3d0        ONLINE       0     0     0
            replacing-4   DEGRADED     0     0     0
              c1t4d0/old  FAULTED      1     6     0  too many errors
              c1t4d0      ONLINE       0     0     0  (resilvering)
            c1t5d0        ONLINE       0     0     0

errors: No known data errors
```

The operation will begin slowly. After some time, the data rate should increase and the estimated time remaining should decrease.

In this example, you can see that member no. 4 is degraded, the old device is faulted, and the new device is online and resilvering.

The resilvering operation will run until it finishes, come what may, unless you explicitly stop it. It even typically survives rebooting, if you must reboot. 

In this case, the Z2 pool has n+2 redundancy, meaning another disk can fail while the first disk is being replaced, without compromising the pool. Still, it would be best to avoid cutting power to the machine or otherwise disrupting it, if only to minimize the opportunities for things to go wrong.

# Investigate further

If you're curious, you can examine the files in /var/adm/messages for any log messages indicating the nature of the failure.

To find out more about the transport errors in these examples, you can grep `/var/adm/messages` for the PCI bus address indicated by cfgadm in the steps above. 

```
[root@node5 ~]# grep pci15d9,f280@1f,2 /var/adm/messages
2017-04-06T20:07:33.816717+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.816751+00:00 node5 scsi: [ID 107833 kern.warning] WARNING: /pci@0,0/pci15d9,f280@1f,2/disk@4,0 (sd5):#012#011SYNCHRONIZE CACHE command failed (5)#012
2017-04-06T20:07:33.816762+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.816837+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.816871+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.816903+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.923190+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.923240+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.923259+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.923322+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.923369+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-06T20:07:33.923379+00:00 node5 sata: [ID 801845 kern.info] /pci@0,0/pci15d9,f280@1f,2:#012 SATA port 4 error
2017-04-10T14:36:36.965224+00:00 node5 sata: [ID 801593 kern.warning] WARNING: /pci@0,0/pci15d9,f280@1f,2:#012 SATA device detected at port 4
```

So it looks like this disk threw at least one error having to do with its cache, along with about ten unspecified errors. 

I'll take it to mean 'bad disk.'
