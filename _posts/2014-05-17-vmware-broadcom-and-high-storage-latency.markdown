---
title: "VMware, broadcom and high storage latency"
layout:     post
header-img: "img/post-bg-01.jpg"
---
We had high storage latency for a long time, and one day I noted a high packet drop, when pinging the vmkernel interface. The physical interface is a broadcom network adapter with multiple interfaces.

So I guessed there was also packet drops for the iSCSI session. But how does one verify this?

The vmkernel has a FreeBSD kernel structure , that can be used to verify there is a high TCP retransmit counters.

```
~ # date ; vsish -e cat /net/tcpip/stats/tcp | grep rexm
Tue Dec 17 20:48:57 UTC 2013
 rexmttimeo:1840
 sndrexmitpack:785
 sndrexmitbyte:681557
 sndrexmitbad:33
 sack_rexmits:37
 sack_rexmit_bytes:53576
 >~ # date ; vsish -e cat /net/tcpip/stats/tcp | grep rexm
 Tue Dec 17 20:49:06 UTC 2013
  rexmttimeo:1905
  sndrexmitpack:785
  sndrexmitbyte:681557
  sndrexmitbad:33
  sack_rexmits:37
  sack_rexmit_bytes:53576
 >~ #
```

In the above we observ 100 retransmit timeouts during 10 seconds.

But after moving the vmnet iSCSI connections to an Intel interface, I can't reproduce any packet drops, even when going from two broadcom interfaces to only one intel interface, and therefor only the half bandwidth.
