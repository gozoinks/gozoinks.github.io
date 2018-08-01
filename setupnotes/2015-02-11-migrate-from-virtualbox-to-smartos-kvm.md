---
title: Migrate from VirtualBox to kvm on SmartOS
slug: migrate-from-virtualbox-to-smartos-kvm
date_published: 2015-02-12T04:19:58.976Z
date_updated:   2015-02-12T04:33:36.099Z
---

Moving Windows Server virtual machines from VirtualBox to kvm seems to have been relatively straightforward. Compatible interfaces helps. Moving the data is a simple matter of converting disk images from VirtualBox to raw and then writing the raw files to a ZVOL.

1. For best results, make sure your VirtualBox guest is using `ide` interfaces for storage and Intel `e1000g` interfaces for networking.
2. On the source host, clone the VirtualBox disk image to a raw format:
`VBoxManage clonehd <source>.vmdk <destination>.img --format raw`
3. Copy it from the source host to the destination host, as with rsync.
4. On the destination host, prepare the vm. The image doesn't really matter. But in the .json file, use `ide` for the hard disk interface and `e1000g` for the networking interface, instead of `virtio`.
5. Stop the vm.
6. Make sure your new vm's `-disk0` zvol is the right size for the image you want to write. If not, use `zfs set` to fix it.
6. Use `qemu-img` to write your `.img` file to the zvol: 
```
qemu-img convert \
-f raw \
-O host_device \
-o size=<gigabytes>G \
<source>.img \
/dev/zvol/rdsk/zones/<uuid>-disk0
```

When that's finished, you should be able to fire up the new vm and pick right up where you left off.

See also:

- [Blue screen BSOD on VirtualBox VM](https://morgansimonsen.wordpress.com/2011/03/12/blue-screen-bsod-on-virtualbox-vm/)
- [Problems with intelppm-sys and processr.sys…](http://blogs.msdn.com/b/virtual_pc_guy/archive/2005/10/25/problems-with-intelppm-sys-and-processr-sys-under-virtual-pc-virtual-server.aspx)
