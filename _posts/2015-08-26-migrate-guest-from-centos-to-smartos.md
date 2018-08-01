---
title: Migrate guest from CentOS to SmartOS
slug: migrate-guest-from-centos-to-smartos
date_published: 2015-08-27T04:52:38.803Z
date_updated:   2015-08-27T04:52:38.801Z
---

It's easy.

First, convert the image on the CentOS host to raw format, if it isn't in that already. SmartOS does not support qcow2 version 3, so you'll have to do the conversion before you make the transfer.

Then, create a vm on the SmartOS host matching the specs of the source machine. I did this manually. You can probably automate it. I was concerned primarily with using "ide" for the primary disk and "e1000" for the primary NIC. Be sure autoboot is off.

Once the image is transferred to the SmartOS host, simply `dd` the .raw image file to the zvol created by vmadm.

Start it up and you should be all set.
