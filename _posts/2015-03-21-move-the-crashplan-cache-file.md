---
title: Move the CrashPlan cache directory
slug: move-the-crashplan-cache-file
date_published: 2015-03-21T22:36:02.766Z
date_updated:   2015-03-21T22:36:02.765Z
---

If you leave the CrashPlan cache file at its default location and your system volume is small, it can fill up your system volume and stop your system.

Moving the cache is as simple as either replacing the default location with a symlink to a larger volume or editing the xml config file.

On a Linux (kvm) virtual machine, the default cache location is /usr/local/crashplan/cache. 

If your cache has already filled up your system disk, you can simply `rm -rf` its contents to clear it.

## Symlink procedure

- A CentOS instance
- CrashPlan data volume mounted at `/crashplan`

```
service crashplan stop
rm -rf /usr/local/crashplan/cache
mkdir -p /crashplan/cache
ln -s /crashplan/cache /usr/local/crashplan/cache
service crashplan start
```

## XML config file procedure

- A CentOS instance
- CrashPlan data volume mounted at `/crashplan`

```
service crashplan stop
rm -rf /usr/local/crashplan/cache
mkdir -p /crashplan/cache
vi /usr/local/crashplan/conf/my.service.xml
# In the <cachePath> element, 
# change the string from /usr/local/crashplan/cache to /crashplan/cache
service crashplan start
```

## References
- [CrashPlan cache files overview](http://support.code42.com/CrashPlan/Latest/Backup/CrashPlan_Cache_Files)
- [Clearing your cache files](http://support.code42.com/CrashPlan/Latest/Troubleshooting/Clearing_Your_Cache_For_Quick_Fixes)
- [Changing CrashPlan cache directory](http://support.code42.com/CrashPlan/Latest/Troubleshooting/Reassigning_Cache_Folder_To_A_Different_Directory)
