---
layout: post
title: "Sysctl source validation"
date: 2017-12-23 00:00:01
---

When a system distracts packets when the route for outbound traffic differs from the route of incoming traffic.

### sysctl.conf
```bash
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.<interface>.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0
```

### /proc
```bash
echo "0" > /proc/sys/net/ipv4/conf/default/rp_filter
echo "0" > /proc/sys/net/ipv4/conf/<interface>/rp_filter
echo "0" > /proc/sys/net/ipv4/conf/all/rp_filter
```

* [Redhat source validation][r1]
[r1]: https://access.redhat.com/solutions/53031
