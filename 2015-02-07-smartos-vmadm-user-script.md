---
title: SmartOS vmadm user-script
slug: smartos-vmadm-user-script
date_published: 2015-02-08T00:12:07.520Z
date_updated:   2015-02-08T00:12:57.808Z
---

SmartOS VMs, at least the ones using JPC images, run the script specified in the user-init property when first provisioned.

This script is run by an SMF service called smartdc/mdata. 

This service has a rather short timeout. 

As a result, if your user-script script takes too long, it will die on startup, and you won't really know why.

The solution is to change the timeout at the start of your user-script script, like so (borrowed from [this script](https://github.com/ZCloud-Firstserver/user-script/blob/master/base64.sh)):

```
if [ ! `svcprop -p start/timeout_seconds mdata:execute` -eq 1800 ]
then
  svccfg -s mdata:execute setprop 'start/timeout_seconds =  count: (1800)'
  svcadm -v refresh mdata:execute
fi
```
