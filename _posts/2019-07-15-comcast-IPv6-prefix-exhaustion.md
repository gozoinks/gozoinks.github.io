---
layout: post
title: Understanding Comcast Business IPv6 prefix delegation
date: 2019-07-15 12:27 -0500
---
Here are my notes from this weekend's IPv6 prefix exhaustion experiment.

Comcast delegates to my router a /59 prefix, instead of a /60. This is unexpected, but it's okay, because one /59 is two /60s. A two-fer.

On the one hand, if you have only one router, this is a limitation; out of the /56 allocated to your account, there are 8 /59s, with only the two /60s per /59. 

## /59 Prefixes in a /56:

|Prefix length |/59 Subnet index |/60 Subnet prefixes |
|--------------|------------|--------------------|
|/56           |1           |00                  |
|              |            |10                  |
|              |2           |20                  |
|              |            |30                  |
|              |3           |40                  |
|              |            |50                  |
|              |4           |60                  |
|              |            |70                  |
|              |5           |80                  |
|              |            |90                  |
|              |6           |A0                  |
|              |            |B0                  |
|              |7           |C0                  |
|              |            |D0                  |
|              |8           |E0                  |
|              |            |F0                  |

You can further assign or delegate neither, either, or both of those /60 subnets, or you can allocate any of the /64s from either of the /60s, so that's still a fair amount of subnets. In a small organization like mine, using a handful of subnets for each of a handful of locations, that’s easily plenty. But for organizations that have a lot more VLANs, like one for each department or team, or many locations, 32 /64s gets a little tight. 

That is, if you have only one router. To get at all those other /59 subnets presumably waiting to be delegated, you need either to request more than one delegation for your one router, or you need more than one router.

To put it another way, your /56 from Comcast supports up to eight routers, with each getting one /59, equal to two /60s or 32 /64s.

# The setup

To see how this works, I wanted to boot up multiple routers all connected to the cable modem, to see what prefixes are delegated to each one and how many would get a delegation. My guess was that the first seven routers would get a delegation, and that the eighth router would either get no delegation or would steal the delegation from the main firewall.

I set up ten pfsense virtual machines in vmware Fusion on my Mac, so that I would have seven routers plus three spares. On the first one, I configured one interface to go to my WAN VLAN, then another interface to go to a “Private to my Mac” VLAN. 

In the pfsense installer on the first VM, I proceeded as far as the “Reboot” screen, then stopped it and duplicated it nine times. Each would boot up, connect to the cable modem, and automatically configure its interfaces. This does result in ten DHCP servers jostling for authority on vmware’s private network, which is bad in production, but of no consequence for this procedure. 

My main router already had been delegated a /59 prefix, ending with e0. That means my delegated subnets include e0 through ff. So the DHCP server seems to be starting at the top of the /56 and working its way down, in which case the next router would get a /59 prefix ending in c0.

# The expected results

Sure enough, the next pfsense instance I started up received a /59 prefix ending in c0. So that’s /64s c0 through df on vm #1, and e0 through ff on my main firewall. 

I repeated this five more times. I got prefixes ending in a0, 80, 60, and 40, just as expected, and each one is two /60s down from the previous one. I expected the sixth one, then, to get a prefix ending in 20. But the sixth one got nothing.

Here's the chart above, reflecting the sequence in which the prefixes were delegated:

## Prefixes delegated

|/59 Subnet index |/60 Subnet prefix   |Delegated to |
|-----------------|:------------------:|------------ |
|8                |E0                  |Main firewall|
|                 |F0                  |Main firewall|
|7                |C0                  |vm1          |
|                 |D0                  |vm1          |
|6                |A0                  |vm2          |
|                 |B0                  |vm2          |
|5                |80                  |vm3          |
|                 |90                  |vm3          |
|4                |60                  |vm4          |
|                 |70                  |vm4          |
|3                |40                  |vm5          |
|                 |50                  |vm5          |
|2                |20                  |             |
|                 |30                  |             |
|1                |00                  |             |
|                 |10                  |             |


Examining the logs on the sixth vm, I saw “status code: no prefixes.” So now we know what happens when you request more than you can get: Comcast sends a notification that no more prefixes can be delegated. That's good!

But then I noticed vm1 no longer had its c0 prefix.

# The unexpected results

I rebooted the vm that had lost its prefix to see if it reacquired a prefix. It did, but he prefix it acquired was e0 — the one that my main router had been assigned. Comcast had taken the prefix from my main router and re-assigned it to the first virtual machine.

I looked back at the main router, and sure enough, IPv6 was down.

I certainly was not expecting /both/ to get "no prefixes" /and/ for my oldest prefix to be revoked and reassigned. /This seems like a bug./

But now, my "production" home network was without IPv6. 

In order to get my original prefix back on the firewall, I would have to convince Comcast's DHCP server to assign that same prefix back to my original firewall's DUID. With all of the prefixes now being allocated, the server was sending "no prefixes" messages. I thought I might try rebooting the main firewall to see if it stole another prefix from one of the VMs, but I wanted to be sure I got the prefix it had before, e0. I didn't want it to be left to chance.

One approach would be to wait for the delegation to expire. A few minutes of searching for the lifetime of a Comcast prefix delegation turned up nothing useful. Being impatient, I decided I would have to tell Comcast to release the prefix I wanted to reclaim. A few minutes of searching on how to tell pfSense to tell a DHCPv6 server to release an address turned up nothing useful beyond going to the Status > Interfaces page and clicking "Release" there.

I wasn't even sure that would work. I've grown accustomed to IPv6 being overlooked in rarely-used functions, and releasing an IPv6 prefix delegation seemed like the kind of thing that might get overlooked. [And it was](https://github.com/pfsense/pfsense/pull/3390). Fortunately, it had been fixed in pfSense 2.4.4. So perhaps this was my ticket.

Because my VM LAN interfaces were all conflicting with each other, I couldn't access the pfsense web configurator page for the VM that had my e0 prefix. Before dealing with that, I tried different ways to trigger a DHCP release from the command line on the console, but pfSense is different enough from plain FreeBSD that dhclient doesn't support the release option. Ultimately, I just shut down all the other VMs so that I could use the web interface.

From there, I went to Status > Interfaces, and then for both the WAN and LAN interfaces, I checked the box to relinquish the lease, then clicked Release. That took down the interface and presumably sent the DHCP release packets. Within a minute or two, my main firewall picked up its e0 delegation again. 

# Wrapping up

So, back to the findings. 

The first firewall got the first /59. I was then able to request five more /59s. That's a total of six /59s Comcast is willing to delegate per customer. 

Why not eight? What about the other two /59s, 00 and 02?

Well, the first /64 (00) is assigned to the cable modem itself, for handing out to directly-connected clients wanting an address in a /64 subnet. That would prevent delegating at least the first /59. It's possible that the cable modem considers the whole 00::/59, including 00::/64 and 01::/64, to belong to itself. That would make sense, if it's following that pattern.

To test that, I picked an address out of 01::/64 and assigned it to the WAN interface on the main router. It works, meaning the cable modem routes that subnet to itself and assumes the address is on its LAN segment, without adding any static or dynamic routes to other hosts.

The second /59, 02::/59, I don't know. Adding addresses in 02::/64 and 03::/64 to the WAN interface on the firewall does not seem to work. So those subnets must be routed elsewhere, perhaps for Comcast's own purposes, such as Comcast's WiFi hotspot service, which I don't use. It seems possible that 02::/64 and 03::/64 are going to that. I will not be testing that here.

# Conclusions

## Multiple firewalls is a new idea

Comcast's approach of delegating a /59 to each device asking for a prefix is unexpected, but provides some benefits. One is the ability to use more than one router behind a given cable modem. Such a thing is hard to imagine from an IPv4 perspective; you might get one or more static IPv4 addresses, but you would have to ask for them and pay extra for them. Being issued a pack of eight /59s and thereby having the ability to run multiple routers at a given site seems decadent.

## Comcast is not communicating well about this

Forums are thick with posts from confused administrators, often new to IPv6, who believe the /56 assigned to their account is fully delegated to their CPE router. It is not. The /56 is delegated to the /cable modem/. Until a CPE router requests a prefix, the cable modem does not have any routes to any of the prefixes in that /56, except apparently for 00 and 01. If you want to use the other subnets within the /56, you have to use prefix delegation. The cable modem will see that you have been delegated a prefix and will automatically add a route to that prefix that points to your CPE. Without that, there is no way to set up a static route from the cable modem to a local device.

## /59 breaks with convention and causes further confusion

Delegating a /59 is a tidy way of giving customers more than one /60 at a time. It's generous and it's sensible, in this writer's opinion. But it breaks with convention. Together with absent documentation, it leads administrators into a world of frustration. 

Convention would be to delegate a /60 prefix, a prefix length four bytes longer than a /56. I believe Comcast did at one time do this, because I am confident that I had routers working with /60 prefix lengths before.

An administrator assuming that Comcast is following this convention is going to configure their routers for a /60 delegation, and when they get a /59, things will break in ways that might be difficult to troubleshoot.

There should be better communication from Comcast about what to expect, and when things change, how things are changing.

# Caveats

Also thick in the forums are threads describing different results with different Comcast cable modems and in different regions. I am given to understand that different areas have different teams managing the IPv6 deployment. I don't know to what extent that is the case, or how much variance that introduces. But it's possible that Comcast is not doing in other towns what it is doing here in Houston, or that a different cable modem would give me different behavior.

I can say that I manage more than one Comcast account in different areas, and that they all have different models of cable modem, and this behavior is the same at all of them.

# TL;DR

"You" don't get a /56 from Comcast, but your cable modem does. And then you can ask your cable modem for up to six /59s.
