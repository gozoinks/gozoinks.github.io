---
title: LSI 1068E stuck at 2TB. Replacing.
slug: replacing-lsi-1068e
date_published: 2015-01-18T20:24:09.488Z
date_updated:   2015-01-18T21:44:25.064Z
layout: post
---

Last night, I discovered that the LSISAS1068E controller in my server under-reports the capacity of my 3-terabyte drives. Apparently, 2TB is the largest capacity it supports. 

I suppose I could have learned that from some documentation, but it never occurred to me to check.

I might not have noticed at all, except for spotting this while troubleshooting an unrelated issue with iostat:
```
iostat -EN
(snip…)
Size: 2199.02GB <2199023254528 bytes>
```
That is, of course, a bit short.

If a firmware update might address this, I can find nothing conclusive about it. The controller is embedded in my SuperMicro motherboard, and [SuperMicro's support page](http://www.supermicro.com/products/motherboard/QPI/5500/X8DT3-F.cfm) doesn't seem to have anything about a firmware update. [LSI's web site](http://www.lsi.com/products/io-controllers/pages/lsi-sas-1068e.aspx#tab/tab4) has nothing about firmware at all. It's probably available on a page for a specific retail product using this chip. Are there even release notes that would mention capacity? Not sure. This is starting to sound like too much work.

There is some discussion about flashing IT mode firmware on these controllers, and [various others](http://lime-technology.com/forum/index.php?topic=12767.0) have gathered up the software it looks like I would need if this were the fix. But even if I do fix the capacity issue, I still have an older, slower controller. It definitely sounds like too much work.

So I have ordered [a new 9211 controller](http://www.neweggbusiness.com/product/product.aspx?item=9b-16-118-114) that not only supports the larger capacity drives out of the box, but also supports 6Gbps links, over the 1068's 3Gbps. 

If I will have to spend the time to reflash firmware, I'd rather reflash to take a new controller from IR mode to IT mode than reflash just to make an old controller work with new disks. And I should be much happier with the performance. Indeed, it was a change I had planned on making eventually anyway. As a bonus, I can get everything I need directly from [LSI's support page for the 9211](http://www.lsi.com/products/host-bus-adapters/pages/lsi-sas-9211-4i.aspx#tab/tab4), rather than from well-meaning but uncertain sources. 

Should've just done all this in the first place.
