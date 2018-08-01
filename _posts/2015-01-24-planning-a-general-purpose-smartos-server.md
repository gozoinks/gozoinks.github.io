---
title: Planning a general-purpose SmartOS server
slug: planning-a-general-purpose-smartos-server
date_published: 2015-01-24T23:48:08.338Z
date_updated:   2015-01-24T23:48:08.335Z
---

After evaluating it together with OpenIndiana and OmniOS, I have decided to run SmartOS on my new general-purpose server at home. 

With that established, it's time to to examine my objectives and to develop a plan to meet those objectives within the design constraints of SmartOS. 

# Objectives
Overall, I want an ample storage server that can host several virtual machines.

## Storage
The storage is mainly for media, photos, projects, and home folders. I want the majority of installed storage available for these purposes. 

## Virtual machines
The virtual machines will host things like Windows Active Directory, Plex, Pydio, a web site or two, and anything I might want to explore over the next few years. 

The overall RAM requirements of these should be low, a few gigs or so.

CPU requirements will be low most of the time, with occasional spikes in demand for things like transcoding.

While storage for system volumes will be private to each VM, many VMs will need to share access to the media directories so that different VMs can offer different services, backed by the same set of data.

## Network
Access to the network by a gigabit Ethernet link should be more than plenty. It is unlikely that clients will demand more than just a few dozen Mbps throughput at any given time. 

Still, I am inclined to aggregate the available links between the server and the switch, and to use VLANs to provide whatever network topology is required by the VMs. I don't care to handle patch cables any more than I have to.

# Constraints
SmartOS is optimized for large-scale cloud deployments. Where there is a trade-off to make, the choice is often made in favor of manageability and automation rather than flexibility and tinkering. In particular, some of its conventions are cumbersome in a smaller deployment.

## Ephemeral global zone
The first and, probably, the most significant constraint of SmartOS is the ephemeral global zone. SmartOS is designed as an operating system for large-scale cloud deployments hosting arbitrarily high numbers of virtual machines. Its design relegates the global zone to minimal duties as a hypervisor and devotes all remaining resources to local zones. 

If no real work happens in the global zone, then I can't do any real work with the kernel. That means I can't rely on the kernel implementations of CIFS/SMB and NFS. For SMB at least, I can use Samba. For NFS, I can only turn to hacks that pierce the veil of the global zone, and I don't want to do that.

## One zpool

A SmartOS system commits all available disks to a single zpool it calls `zones`. Not every dataset on this pool is actually a zone, so this is an unfortunately inaccurate name. 

The system assumes that this is where all zone data should go, so it can't have any other name.

Being the only place to store persistent data, zone or otherwise, this is where I will have to put any datasets I want to share among multiple VMs.

I'll just have to be careful when managing datasets, to be sure I don't break a zone or blow away a dataset I want to keep.

# Solutions

Well, let's see how it goes.
