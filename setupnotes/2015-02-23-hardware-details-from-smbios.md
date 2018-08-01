---
title: Hardware details from smbios
slug: hardware-details-from-smbios
date_published: 2015-02-23T21:12:21.787Z
date_updated:   2015-02-23T21:12:21.786Z
---

To get detailed data about installed hardware, including CPU model and speed, RAM capacity and memory speed, device model numbers and serial numbers, and just about everything else, the tool you want is:
```
smbios
```
This tool serves the role of what in Linux would be `dmidecode` and displays a great deal more info than the usual Solaris-world tools of `prtconf` and `prtdiag`. 

With it, you can get exact part numbers for installed RAM modules, for example. You can also see what UUID has been assigned to your logic board, assuming that was done properly.
