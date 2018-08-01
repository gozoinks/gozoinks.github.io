---
title: Saltstack on SmartOS
slug: saltstack-on-smartos
date_published: 2015-03-09T02:02:49.790Z
date_updated:   2015-03-09T02:02:49.789Z
---

Packages are available from Joyent for both salt-master and salt-minion (packaged together as simply 'salt'). 

Service manifests are not provided.

Salt comes preconfigured with 'loghost' for a hostname.

Not including the configuration steps you'll need to take, this is the procedure.

Install on server or on client:
```
pkgin -y install salt
```

Start the server:
```
salt-master -d
```

Fix the minion_id on the client:
```
echo `hostname` > /opt/local/etc/salt/minion_id
```

Start the client:
```
salt-minion -d
```

Accept the client's key on the server:
```
salt-key -L
salt-key -A
```

When configuring the server, use cryptpass to generate the password:
```
/usr/lib/cryptpass "$password"
```
