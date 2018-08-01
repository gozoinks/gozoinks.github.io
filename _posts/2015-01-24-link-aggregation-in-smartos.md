---
title: Link aggregation in SmartOS
slug: link-aggregation-in-smartos
date_published: 2015-01-25T00:25:43.536Z
date_updated:   2015-01-25T00:58:54.008Z
---

Link aggregation in SmartOS is documented here:

https://wiki.smartos.org/display/DOC/Managing+NICs#ManagingNICs-LinkAggregationsintheGlobalZone

The admin network cannot be on a VLAN between the port and the interface. There's some muttering about it on Github and in IRC logs, but it does not now nor has it ever worked. The admin network must be configured in access mode on the port(s) connecting the server to the switch. Other networks can be tagged on that link, but the admin network must be untagged.

Therefore, if you aggregate all the interfaces on a node, then the admin LAN must be the native LAN for the aggregated interface. The switch must untag the admin VLAN on the aggregated ports.

Other VLANs can certainly be tagged. When defining the vnics in `/usbkey/config`, simply add a `_vlan_id` parameter.
