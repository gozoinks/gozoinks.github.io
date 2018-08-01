---
title: Monitor APC UPS with Monit
slug: monitor-apc-ups-with-monit
date_published: 2017-09-15T21:48:15.519Z
date_updated:   2017-09-15T21:52:56.813Z
---

Monitor your UPS with [monit](http://mmonit.com/monit) and [apcupsd](http://www.apcupsd.org), using an `sh` one-liner.

Example monit config:
```
check program ups with path "/usr/bin/sh -c 'STATUS=`/opt/local/sbin/apcaccess -p STATUS`; OK='ONLINE'; if [ $STATUS != $OK ]; then echo $STATUS; exit 1; fi;'"
  if status != 0 then alert
```
Explanation:

- `/usr/bin/sh -c` — execute the argument string in the bourne shell. 
- `/opt/local/sbin/apcaccess -p STATUS` — get the value of the 'STATUS' field returned by `apcaccess`
- `if [ $STATUS != $OK ]; then echo $STATUS; exit 1; fi;` — compare the result to what you want; if there's no match, then echo what you did get, so that Monit can report that in the alert, then exit with status 1 so that Monit knows there is a problem.
- `if status != 0 then alert` — back in monit land, if the exit value is not 0, then alert.
