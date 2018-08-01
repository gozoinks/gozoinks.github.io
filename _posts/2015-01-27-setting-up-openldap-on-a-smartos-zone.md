---
title: Setting up OpenLDAP on a SmartOS zone
slug: setting-up-openldap-on-a-smartos-zone
date_published: 2015-01-28T04:58:58.022Z
date_updated:   2015-01-30T02:03:31.190Z
---

1. Install: `pkgin install openldap`
2. Configure: `vi /opt/local/etc/slapd.conf`
3. Get this SMF manifest, save it as, say, `slapd.xml`: https://gist.github.com/apung/c045c6c3db1df613476d
4. Fix the SMF manifest; in the start method, change `/opt/local/sbin/slapd` to `/opt/local/libexec/slapd`
5. Import the gist: `svccfg import slapd.xml`
6. Start it up: `svcadm enable slapd`

I find that it takes a second to start up before `svcs -xv` comes back clean.

Now use Apache Directory Studio to connect, using the credentials you put into the configuration file.

#### To do: ####

- Hash the admin password
- Initialize the directory so I can start putting accounts into it

