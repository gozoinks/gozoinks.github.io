---
title: Avoid needing to override ashift with SmartOS
slug: smartos-makes-overriding-ashift-difficult
date_published: 2015-01-24T19:15:58.949Z
date_updated:   2015-01-24T19:15:58.947Z
---

The Illumos way of correcting physical sector size on a new zpool is to edit `/kernel/drv/sd.conf` and reboot. 

With SmartOS, edits to `sd.conf` do not persist between reboots, so you can't exactly do that.

To work around this, make your changes to sd.conf and use `update_drv` to reload the SCSI driver and apply the changes. Create your zpool before you reboot. Once created, the correct zpool ashift value will stick.

Ultimately, though, the solution is to use disks with 4K sectors that correctly report their 4K sector size, connected to a controller (and backplane, if applicable) that also correctly reports their 4K sector size. SmartOS does the right thing if those planets are aligned. It's far better to do it this way, and with modern controllers and modern disks, you should be much happier.
