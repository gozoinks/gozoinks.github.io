---
title: Enable IPv6 in SmartOS global zone
slug: enable-ipv6-in-smartos
date_published: 2015-10-11T22:49:19.621Z
date_updated:   2015-10-11T22:49:28.768Z
---

```
# Get the name of the link you want to enable IPv6 on:
dladm show-link

# Enable neighbor discovery protocol:
svcadm enable ndp

# Create the IPv6 interface:
ipadm create-addr -t -T addrconf aggr0/v6

# Confirm the results:
ipadm show-addr
```
