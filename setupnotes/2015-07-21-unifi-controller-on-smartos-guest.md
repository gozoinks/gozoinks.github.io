---
title: UniFi controller on SmartOS guest
slug: unifi-controller-on-smartos-guest
date_published: 2015-07-21T21:07:27.682Z
date_updated:   2015-07-21T21:07:27.681Z
---

This works with the provided mongodb and openjdk7 packages. Mongodb depends on LC_ALL being set to 'C', which is not the default.

After symlinking /opt/local/bin/mongod to /opt/local/UniFi, there is no need to enable the mongodb service using SMF; UniFi's ace.jar does it for you (via the symlink).

1. Prepare a SmartOS vm. 2GB RAM is recommended.
2. Install dependencies: ```pkgin -y install openjdk7 mongodb```
3. Download the UniFI distribution: ```curl -OL http://dl.ubnt.com/unifi/4.6.6/UniFi.unix.zip```
4. Unzip the UniFi controller into the /opt/local directory: ```/opt/local/unzip -o Unifi.unix.zip -d /opt/local```
5. Fix UniFi's mongodb link: ```cd /opt/local/UniFi/bin && ln -s /opt/local/bin/mongod mongod```
6. Use manifold to create a unifi.xml manifest file for SMF, then start it with ```svcadm enable unifi```.
