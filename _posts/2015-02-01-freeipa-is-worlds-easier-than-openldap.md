---
title: FreeIPA is worlds easier than OpenLDAP
slug: freeipa-is-worlds-easier-than-openldap
date_published: 2015-02-02T03:13:15.899Z
date_updated:   2015-02-02T03:19:21.507Z
---

I've been able to get FreeIPA up and running in far less time and with far less frustration than OpenLDAP.

The notes [here](http://linsec.ca/Using_FreeIPA_for_User_Authentication) were particularly helpful.

I've set this up in a CentOS KVM zone with very little trouble.

Be sure to set the IP address, gateway, and DNS resolvers manually. If you rely on DHCP and the configuration from the global zone, dhclient will clobber the changes FreeIPA depends on.

Here is an abridged VM .json file and an abridged installation script.

```
{
  "brand": "kvm",
  "ram": "512",
  "vcpus": 1,
  "alias": "ipa01",
  "hostname": "ipa01.mydomain.net",
  "resolvers": [
    "8.8.8.8",
    "4.2.2.1"
  ],
  "dns_domain": "mydomain.net",
  "nics": [
    {
      "nic_tag": "admin",
      "ip": "10.91.0.206",
      "netmask": "255.255.0.0",
      "gateway": "10.91.0.1",
      "model": "virtio"
    }
  ],
  "disks": [
    {
      "image_uuid": "5becfd74-a70d-11e4-93a6-470507be237c",
      "boot": true,
      "model": "virtio"
    }
  ],
  "customer_metadata": {
    "root_authorized_keys": "<my ssh public key>",
    "user-script": "/usr/sbin/mdata-get root_authorized_keys > ~root/.ssh/authorized_keys ; /usr/sbin/mdata-get root_authorized_keys > ~admin/.ssh/authorized_keys"
  }
}
```

Installation script:

```
yum update -y
yum install -y ipa-server bind bind-dyndb-ldap
echo "10.91.0.206 ipa01.mydomain.net" >> /etc/hosts
ipa-server-install -a <password> --hostname=ipa01.mydomain.net -n mydomain.net -p <password> -r MYDOMAIN.NET --setup-dns --forwarder=8.8.8.8 --forwarder=4.2.2.2
service sshd restart

```
Remember to make your IP address, gateway, and especially resolvers static.
