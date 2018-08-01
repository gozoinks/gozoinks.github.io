---
title: SATA target names and SAS worldwide names
slug: controller-names-and-worldwide-names
date_published: 2015-01-24T19:59:09.281Z
date_updated:   2015-01-24T20:00:26.010Z
---

If you've worked only with older LSI controllers like the 1068 on Solaris-like systems, you'll be familiar with the `c0t0d0` convention for naming devices connected to your storage controller, typical of SATA disks connected to SATA controllers.

As soon as you encounter a newer controller like the LSI 2000-series, however, you may instead encounter the [SAS "Worldwide Name" or WWN](http://en.wikipedia.org/wiki/World_Wide_Name), which is a bit more complicated than the older target names.

You'll see this when you do an `iostat -En` on the newer controller. Where you would once have seen something like this on the older controller:
```
c0t5d0 Soft Errors: 0 Hard Errors: 0 Transport Errors: 0 
```
Instead you'll see something like this on the newer controller:
```
c0t50014EE26055AF5Dd0 Soft Errors: 0 Hard Errors: 0 Transport Errors: 0 
```
This happens even on SATA disks, if they're connected to a SAS controller or backplane.

The longer target name is the SAS WWN of the disk, which is designed to be unique to the disk regardless of the topology of the connection. It's common with SAS systems, so this will be old news to someone more accustomed to working exclusively with SAS equipment. But if you're new to SAS, or if you're working with SATA disks on SAS controllers, this is a bit of a surprise.

Unfortunately, it means you'll be typing much more complicated things when it's time to create a zpool. 

So I hope you have your serial-over-lan console working, because you're going to want to copy and paste in your terminal window.
