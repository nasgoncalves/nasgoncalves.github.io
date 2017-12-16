---
layout: post
title: "Docker and Systemd"
date: 2017-12-16 00:00:00
---

### Dockerfile

```dockerfile
FROM centos/systemd

LABEL mantainer="nuno.alberto.soares.goncalves@gmail.com"

RUN yum -y install chrony; yum clean all; systemctl enable chronyd.service

CMD ["/usr/sbin/init"]
```

### Build

```console
docker build --rm --no-cache -t nasgoncalves/centos7-systemd-chronyd .
```

### Run

```console
docker run --privileged --name centos7-baseline -v /sys/fs/cgroup:/sys/fs/cgroup:ro -d nasgoncalves/centos7-systemd-chronyd
docker exec -it 8d1936faf8c8 /bin/bash
```

### Checking

```console
[root@8d1936faf8c8 /]# systemctl status chronyd.service
● chronyd.service - NTP client/server
   Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2017-12-16 18:44:23 UTC; 16min ago
     Docs: man:chronyd(8)
           man:chrony.conf(5)
  Process: 22 ExecStartPost=/usr/libexec/chrony-helper update-daemon (code=exited, status=0/SUCCESS)
  Process: 18 ExecStart=/usr/sbin/chronyd $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 20 (chronyd)
   CGroup: /docker/8d1936faf8c8b1e508c17ed86783880b4f3f7373450e561c102022f9db322313/system.slice/chronyd.service
           └─20 /usr/sbin/chronyd

Dec 16 18:44:23 8d1936faf8c8 systemd[1]: Starting NTP client/server...
Dec 16 18:44:23 8d1936faf8c8 chronyd[20]: chronyd version 3.1 starting (+CMDMON +NTP +REFCLOCK +RTC +PRIVDROP +SCFILTER +SECH...+DEBUG)
Dec 16 18:44:23 8d1936faf8c8 systemd[1]: Started NTP client/server.
Dec 16 18:44:28 8d1936faf8c8 chronyd[20]: Selected source 178.79.152.182
Dec 16 18:45:34 8d1936faf8c8 chronyd[20]: Selected source 134.0.16.1
Dec 16 18:46:48 8d1936faf8c8 chronyd[20]: Source 213.251.52.107 replaced with 85.199.214.100
Hint: Some lines were ellipsized, use -l to show in full.
[root@8d1936faf8c8 /]# watch -n1 date
[root@8d1936faf8c8 /]# date
Sat Dec 16 19:00:56 UTC 2017
[root@8d1936faf8c8 /]# chronyc sources
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^- ntp1.wirehive.net             2   6    17    60    -42ms[  -42ms] +/-   77ms
^* lond-web-1.speedwelshpoo>     2   6    17    60   +726us[+3937us] +/-   42ms
^- duke.v4.colo.m.faelix.net     2   6    17    60  +2083us[+2083us] +/-   54ms
^? 213.251.52.107                0   7     0     -     +0ns[   +0ns] +/-    0ns
```

* [centos/systemd][r1]

[r1]: https://hub.docker.com/r/centos/systemd/
