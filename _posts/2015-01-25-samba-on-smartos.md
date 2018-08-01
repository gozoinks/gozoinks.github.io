---
title: Samba on SmartOS
slug: samba-on-smartos
date_published: 2015-01-26T02:55:39.295Z
date_updated:   2015-01-26T02:55:39.293Z
---

The following article is more current than what typically comes up in a search for "samba on smartos."

http://www.cyber-tec.org/2015/01/09/setting-up-samba-on-smartos/

Some changes since the original Jonathan Perkin article:

- Samba is included in the main pkgsrc repository
- SMF manifests are included

In short:

1. `pkgin -y install samba`
2. Edit `/opt/local/etc/samba/smb.conf` for your installation
3. `svcadm enable svc:/pkgsrc/samba:nmbd`
4. `svcadm enable svc:/pkgsrc/samba:smbd`
5. `svcadm enable svc:/network/dns/multicast:default`

