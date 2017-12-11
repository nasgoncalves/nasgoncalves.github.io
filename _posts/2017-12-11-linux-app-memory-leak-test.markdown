---
layout: post
title: "Linux Application Memory Leak Test"
date: 2017-12-11 00:00:00
---

### With Deamon

```
valgrind --leak-check=full --log-file="logfile.out" --trace-children=yes --track-origins=yes -v ./splunk start
```

### Without Deamon

```
valgrind --leak-check=full --log-file="logfile.out" -v ./splunk start
```

### Results

```
==26829==  Uninitialised value was created by a stack allocation
==26829==    at 0x125FED0: AesCcm192NonceGenerator::AesCcm192NonceGenerator() (in /data/splunk/bin/splunkd)

==26829==  Uninitialised value was created by a stack allocation
==26829==    at 0xA31AD6: FileInputTracker::fileHalfMd5(unsigned long*, FileDescriptor, Str const&, long, long) (in /data/splunk/bin/splunkd)

==26829==  Uninitialised value was created by a stack allocation
==26829==    at 0x126054D: AesCcm192Key::encrypt(StrSegment const&, AesCcm192Nonce const&, Str const&) const (in /data/splunk/bin/splunkd)
```