---
layout: post
title: Un-yank a yanked disk
date: 2018-08-05 12:00 -0500
---
If you, a toddler, or a toddler working for you pulls a disk from a pool unexpectedly, the pool will move to a "DEGRADED" state, and a `zpool status` will advise you to take some more drastic steps, like replacing the device (which reformats the disk and resilvers from parity). 

To recover somewhat more gracefully, you can try putting the disk back online and scrubbing the pool, which should correct any errors caused by disconnecting a disk while I/O operations are active.

# To resume a disk taken offline unexpectedly:
1. Reconnect the disk. Wait for it to spin up, if necessary.
2. `zpool online <pool> <device ID>`. For example: `zpool online zones c0t50014EE05910B3B1d0`
3. `zpool scrub <pool>`. For example: `zpool scrub zones`
4. Wait however many hours it takes to scrub your pool. You should end up seeing some errors corrected.
5. If you still see warnings in the output of `zpool status` after the scrub, do a `zpool clear` on the affected device, then `zpool scrub` again.
6. If, after all this, you're still seeing errors, _then_ go through the `zpool replace` procedure on the failed disk. From here on out, you may be dealing with a more major fault, so be prepared to handle it more as you would a disk failure.