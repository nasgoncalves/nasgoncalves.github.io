---
layout: post
title: "Performance Metrics & Splunk"
date: 2018-01-13 00:00:02
---

### Install & configure collectd

```bash
yum install -y epel-release
yum install -y collectd policycoreutils-python
setsebool -P collectd_tcp_network_connect 1
semanage permissive -a collectd_t
service collectd restart
```

### /etc/collectd.conf
```
FQDNLookup false
LoadPlugin logfile
LoadPlugin cpu
LoadPlugin df
LoadPlugin disk
LoadPlugin interface
LoadPlugin load
LoadPlugin memory
LoadPlugin processes
LoadPlugin protocols
LoadPlugin swap
LoadPlugin tcpconns
#LoadPlugin thermal
LoadPlugin uptime

<Plugin memory>
  ValuesAbsolute true
  ValuesPercentage true
</Plugin>

<Plugin swap>
  ReportByDevice true
  ValuesPercentage true
</Plugin>

<Plugin df>
  ReportByDevice true
  ValuesPercentage true
</Plugin>

<Plugin logfile>
  LogLevel info
  File "/var/log/collectd.log"
  Timestamp true
  PrintSeverity false
</Plugin>

<Plugin load>
  ReportRelative true
</Plugin>

<Plugin processes>
  ProcessMatch "all" "(.*)"
</Plugin>

<Plugin cpu>
  ValuesPercentage true
</Plugin>

<Plugin interface>
  Interface "lo"
  IgnoreSelected false
  ReportInactive true
  UniqueName false
</Plugin>

<Plugin protocols>
  Value "/^Tcp:/"
  IgnoreSelected false
</Plugin>

<Plugin tcpconns>
  ListeningPorts true
  AllPortsSummary true
</Plugin>

```

# Collectd to Splunk HEC

# Workflow
[System] -> [Splunk HF HEC] -> [Splunk Indexer]

## System Configuration

### /etc/collectd.d/hec.conf

```
LoadPlugin write_http

<Plugin write_http>
  <Node "HEC">
    URL "https://<splunk-hec>:8088/services/collector/raw"
    Header "Authorization: Splunk <token>"
    Format "JSON"
    VerifyPeer false
    VerifyHost false
    Metrics true
    StoreRates true
  </Node>
</Plugin>
```

## HF HEC Configuration

### inputs.conf

```
[http]
port = 8088
disabled = 0

[http://collectd]
token = <guid>
disabled=0
index=collectd
source=collectd token
sourcetype=collectd_http
outputgroup = INDEXER
```

### outputs.conf

```
[indexAndForward]
index = false

[tcpout:INDEXER]
server = <indexer_ip>:<indexer_port>
useACK=false
maxQueueSize = 50MB
```

## Indexer Configuration

### inputs.conf

```
[splunktcp://9997]
index =  collectd
```

### indexers.conf

```
[collectd]
datatype   = metric
homePath   = $SPLUNK_DB/collectd/db
coldPath   = $SPLUNK_DB/collectd/colddb
thawedPath = $SPLUNK_DB/collectd/thaweddb
```

<br>

# Collectd to Splunk TCP

## Workflow
[System] -> [Splunk HF TCP] -> [Splunk Indexer]

<br>

## System Configuration

### /etc/collectd.d/tcp.conf

```
LoadPlugin write_graphite

<Plugin write_graphite>
  <Carbon>
    Host "<splunk-hf>"
    Port "9997"
    Protocol "tcp"
  </Carbon>
</Plugin>
```

## HF TCP Configuration

### inputs.conf

```
[tcp://9997]
index = collectd
sourcetype = graphite_collectd
outputgroup = INDEXER
```

### outputs.conf

```
[indexAndForward]
index = false

[tcpout:INDEXER]
server = <indexer_ip>:<indexer_port>
useACK=false
maxQueueSize = 50MB
```

## Indexer Configuration

### inputs.conf

```
[splunktcp://9997]
index =  collectd
```

### indexers.conf

```
[collectd]
homePath   = $SPLUNK_DB/collectd/db
coldPath   = $SPLUNK_DB/collectd/colddb
thawedPath = $SPLUNK_DB/collectd/thaweddb
```

### props.conf

```
[graphite_collectd]
TIME_PREFIX = ^.+\..+\..+\s.+\s
EXTRACT-metric_value = ^(?P<host>[^\.]+)[^\.\n]*\.(?P<object>[^\.]+)\.(?P<metric>[^ ]+)\s+(?P<value>[^ ]+)
EXTRACT-metric_kv = ^(?P<host>[^\.]+)[^\.\n]*\.(?P<object>[^\.]+)\.(?P<_KEY_1>[^ ]+)\s+(?P<_VAL_1>[^ ]+)
SHOULD_LINEMERGE=false
TRANSFORMS-mask1= mask-collectd1
TRANSFORMS-mask2= mask-collectd2
TRANSFORMS-mask3= mask-collectd3
TRANSFORMS-mask4= mask-collectd4
TRANSFORMS-mask5= mask-collectd5
TRANSFORMS-mask6= mask-collectd6
TRANSFORMS-mask7= mask-collectd7
TRANSFORMS-mask8= mask-collectd8
TRANSFORMS-mask9= mask-collectd9
EVAL-host = replace(host,"_",".")

```

### transforms.conf

```
[mask-collectd1]
REGEX = ^([^_]+)\..+\..+\s.+\s.+
DEST_KEY = MetaData:Host
FORMAT =  host::$1

[mask-collectd2]
REGEX = ^([^_]+)_([^_.]+)\..+\..+\s.+\s.+
DEST_KEY = MetaData:Host
FORMAT =  host::$2.$1

[mask-collectd3]
REGEX = ^([^_]+)_([^_]+)_([^_.]+)\..+\..+\s.+\s.+
DEST_KEY = MetaData:Host
FORMAT =  host::$3.$2.$1

[mask-collectd4]
REGEX = ^([^_]+)_([^_]+)_([^_]+)_([^_.]+)\..+\..+\s.+\s.+
DEST_KEY = MetaData:Host
FORMAT =  host::$4.$3.$2.$1

[mask-collectd5]
REGEX = ^([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_.]+)\..+\..+\s.+\s.+
DEST_KEY = MetaData:Host
FORMAT =  host::$5.$4.$3.$2.$1

[mask-collectd6]
REGEX = ^([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_.]+)\..+\..+\s.+\s.+
DEST_KEY = MetaData:Host
FORMAT =  host::$6.$5.$4.$3.$2.$1

[mask-collectd7]
REGEX = ^([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_.]+)\..+\..+\s.+\s.+
DEST_KEY = MetaData:Host
FORMAT =  host::$7.$6.$5.$4.$3.$2.$1

[mask-collectd8]
REGEX = ^([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_.]+)\..+\..+\s.+\s.+
DEST_KEY = MetaData:Host
FORMAT =  host::$8.$7.$6.$5.$4.$3.$2.$1

[mask-collectd9]
REGEX = ^([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_]+)_([^_.]+)\..+\..+\s.+\s.+
DEST_KEY = MetaData:Host
FORMAT =  host::$9.$8.$7.$6.$5.$4.$3.$2.$1
```

<br>

# Bonus: Collectd to Splunk CSV monitor

## Workflow
[System] -> [Splunk HF Monitor] -> [Splunk Indexer]

### collectd csv plugin (/usr/local/lib/collectd/python/python_csv_plugin.py)

```python
import collectd

COLLECTD_METRCIS_LOG = "/var/log/collectd-metrics.log"

def config(ObjConfiguration):
    collectd.debug('Configuring')

def init():
    collectd.debug('Initialization')

def write(vl, data=None):

      row = ["%s,%s,%s,%s,%s,%s\n" % \
                (vl.time, vl.host, vl.plugin, \
                vl.type, vl.plugin_instance, value) \
                for value in vl.values]

      with open(COLLECTD_METRCIS_LOG, 'a+') as csvfile:
          for r in row:
              csvfile.write(r)

collectd.register_config(config)
collectd.register_init(init)
collectd.register_write(write)
```

### /etc/collectd.d/csv.conf

```
LoadPlugin python

<Plugin python>
  ModulePath "/usr/local/lib/collectd/python"
  LogTraces true
  Interactive false
  Import python_csv_plugin
  <Module python_csv_plugin>
  </Module>
</Plugin>
```

Log Sample (/var/log/collectd-metrics.log):

```
1515962589.16,system,processes,ps_rss,collectd,12378112.0
1515962589.16,system,processes,ps_vm,collectd,1015365632.0
1515962589.16,system,processes,ps_stacksize,collectd,2176.0
1515962589.16,system,processes,ps_cputime,collectd,600000
1515962589.16,system,processes,ps_cputime,collectd,1150000
1515962589.16,system,processes,ps_code,collectd,15937536.0
1515962589.16,system,processes,ps_pagefaults,collectd,11078
1515962589.16,system,processes,ps_pagefaults,collectd,0
1515962589.16,system,processes,io_octets,collectd,2785618
1515962589.16,system,processes,io_octets,collectd,503948
```

<br>

# Refs

* [Collectd App for Splunk Enterprise](https://splunkbase.splunk.com/app/2875/)
* [Configure Collectd to send data to Splunk](http://docs.splunk.com/Documentation/AddOns/released/Linux/Configure)
* [Getting Data from Collectd or Graphite into Splunk](https://www.splunk.com/blog/2013/07/26/getting-data-from-collectd-into-splunk.html)
* [Analytics for Linux](https://splunkbase.splunk.com/app/3777/)
