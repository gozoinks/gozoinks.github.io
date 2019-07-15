---
layout: post
title: Comcast Business delegates IPv6 /59s
date: 2019-07-10 10:52 -0500
---
Reports differ on what size prefix Comcast delegates to Business Internet (coax) customers. In my home market of Houston, Texas, I seem to be delegated a /59 prefix, not a /60 as I had expected. 

If dhcp6c is configured to expect a /60, but actually receives a /59, it appears to configure LAN interfaces with a prefix length of /63 instead of /64. This prevents SLAAC from working and causes routing issues. The solution is to make sure dhcp6c's prefix length matches what is actually delegated.

Prefix hints seem to be ignored by Comcast DHCP servers. You will receive what Comcast's DHCP server chooses to give you, regardless of what you request. So sending a prefix hint is not apparently necessary.

Specifying the correct prefix length, however, appears to be quite necessary. It doesn't have any effect on the DHCP server, but it does affect the way dhcp6c calculates the subnet size for the prefixes it receives and then allocates to LAN interfaces.

So: when configuring a firewall with Comcast Business IPv6, it may be best to check the dhcp6c debug logs for an indication of the prefix length you are actually being delegated. Shortly after some lines about IA_PD, there should be a line indicating your prefix and its length. Then go back and configure your dhcp6c client to expect that same prefix length.
