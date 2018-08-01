---
title: UniFi Video on SmartOS via Debian
slug: unifi-video-on-smartos-via-debian
date_published: 2015-06-13T05:06:13.985Z
date_updated:   2015-06-13T05:06:13.984Z
---

```
# Install a Debian 7 image:
imgadm import 2d1df15c-d2ef-11e4-b9dd-1f436f2c33c9

# Configure a Debian 7 vm
...

# Create a zvol for video
zfs create -V 1T zones/<uuid>-disk1

# Add it to the VM:
vi nvr-add-disk1.json 
{
  "add_disks": [
    {
      "nocreate": true,
      "boot": false,
      "model": "virtio",
      "media": "disk",
      "path": "/dev/zvol/rdsk/zones/<uuid>-disk1",
      "size": 1099510
    }
  ]
}

vmadm update <uuid> -f nvr-add-disk1.json

# Log in and format it

http://www.debiantutorials.com/how-to-add-a-new-hard-disk-or-partition-using-uuid-and-ext4-filesystem/

# Install the UniFi NVR software .deb: 
curl -OL http://dl.ubnt.com/firmwares/unifi-video/3.1.1/unifi-video_3.1.1~Debian7_amd64.deb

http://community.ubnt.com/t5/UniFi-Video-Blog/UniFi-Video-3-1-1-Release/ba-p/1215852

```
