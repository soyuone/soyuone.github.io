---
layout: post
title: "Elasticsearch概述"
categories: 大数据
tags: 大数据 Elasticsearch Kibana
author: Kopite
---

* content
{:toc}


本文中的`Elasticsearch`概述等相关内容翻译、总结自官网[Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/index.html)。



## 节点

启动一个`Elasticsearch`实例即启动一个`Elasticsearch`节点，相连接节点的集合称为`Elasticsearch`集群，另外每个节点都有一个或多个角色：
* `Master-eligible node`，`node.master`设置为`true`（默认值）会将节点设置为候选主节点，主节点用于控制集群。
* `Data node`，`node.data`设置为true（默认值）会将节点设置为数据节点，数据节点保存数据并执行与数据有关的操作，例如CRUD、检索和聚合。
* `Ingest node`，`node.ingest`设置为true（默认值）会将节点设置为`ingest node`。`ingest node`能够将[ingest pipeline](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/pipeline.html)应用于文档，以便在建立索引之前转换和丰富文档。在繁重的`ingest`负载下，最好使用`dedicated ingest nodes`并将`master`节点、`data`节点配置为`node.ingest：false`。
* `Tribe node`，通过tribe。*设置配置的部落节点是仅协调节点的一种特殊类型，它可以连接到多个集群并在所有连接的集群中执行搜索和其他操作。

默认情况下，一个节点是一个符合资格的主节点和一个数据节点，此外，它还可以通过摄取管道对文档进行预处理。这对于小型集群非常方便，但是随着集群的增长，考虑将专用于主机的专用节点与专用数据节点分离就变得很重要。

### Master-eligible node

`master node`负责集群范围内的轻量级操作，比如创建或删除索引、跟踪哪些节点是集群的一部分以及确定将哪些分片分配给哪些节点，拥有稳定的`master node`对于集群的健康非常重要。
<br>
<br>
索引和搜索文档会占用大量`CPU`、`内存`和`I/O`，这可能给节点的资源带来压力。为确保`master node`稳定且不受压力，在较大的集群中，最好将`master-eligible node`和专用`data node`分开。
<br>
<br>
虽然`master node`还可以充当`coordinating nodes`，并将搜索和索引请求从客户端路由到`data node`，但最好不要基于此目的而使用`master node`。对于`master-eligible node`，其工作量应尽可能少，这对于集群的稳定性很重要。创建`dedicated master-eligible node`，需要做如下配置：

```
node.master: true 
node.data: false 
node.ingest: false 
search.remote.connect: false 
```

#### 避免脑裂

为避免脑裂现象，组成集群时的可用`discovery.zen.minimum_master_nodes`数量需要大于等于`(master_eligible_nodes / 2) + 1`，该配置项默认值为1。
<br>
<br>
在专用节点之间划分`master-eligible node`角色和`data node`角色的优点之一是，可以只有三个`master-eligible node`，并将`minimum_master_nodes`设置为2，之后无论将多少个专用`data node`添加到集群，都不必再更改此配置。

### Data node

`data node`存放索引分片，并处理与数据相关的操作，例如`CRUD`，搜索和聚合。这些操作会占用`I/O`，`内存`和`CPU`，监视这些资源的占用情况并在过载时添加更多的`data node`非常重要。
<br>
<br>
设置`dedicated data nodes`的主要好处是将`master`角色和`data`角色分开，创建`dedicated data node`需要做如下配置：

```
node.master: false 
node.data: true 
node.ingest: false 
search.remote.connect: false 
```

### Ingest Node

在实际文档建立索引之前，请使用摄取节点对文档进行预处理。接收节点拦截批量和索引请求，应用转换，然后将文档传递回索引或批量API。



索引模块控制与索引相关的设置，这些设置针对所有索引进行全局管理，而不是在每个索引级别进行配置，可进行如下配置：
* [Circuit breaker](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/circuit-breaker.html)
* [Fielddata cache](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/modules-fielddata.html)
* [Node query cache](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/query-cache.html)
* [Indexing buffer](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/indexing-buffer.html)
* [Shard request cache](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/shard-request-cache.html)
* [Recovery](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/recovery.html)

### Fielddata



### Indexing Buffer

`Indexing Buffer`用于存储新索引的文档，该缓冲区填满后，其缓冲的文档将被写到磁盘的一个段中，`Indexing Buffer`由在该节点上的所有分片共享。
<br>
<br>
以下为静态配置，并且必须在集群中的每个`data node`上进行配置：
* `indices.memory.index_buffer_size`，接受百分比或字节大小的值，默认值为该节点`heap`内存大小的10％，由该节点上的所有分片共享。
* `indices.memory.min_index_buffer_size`，如果`index_buffer_size`设置为百分比，则此参数可用于指定绝对最小值，默认为48mb。
* `indices.memory.max_index_buffer_size`，如果`index_buffer_size`指定为百分比，则此参数可用于指定绝对最大值，默认为无界。



```
[root@localhost local]# ll
总用量 283460
drwxr-xr-x.  2 root   root           6 8月  12 2015 bin
-rw-r--r--.  1 root   root   290094012 1月   7 14:43 elasticsearch-7.5.1-linux-x86_64.tar.gz
drwxr-xr-x.  2 root   root           6 8月  12 2015 etc
drwxr-xr-x.  2 root   root           6 8月  12 2015 games
drwxr-xr-x.  2 root   root           6 8月  12 2015 include
drwxr-xr-x.  7 root   root          94 11月 19 2018 kafka_2.12-2.0.0
drwxr-xr-x.  2 root   root           6 8月  12 2015 lib
drwxr-xr-x.  2 root   root           6 8月  12 2015 lib64
drwxr-xr-x.  2 root   root           6 8月  12 2015 libexec
drwxr-xr-x.  2 root   root           6 8月  12 2015 sbin
drwxr-xr-x.  5 root   root          46 11月 15 2018 share
drwxr-xr-x.  2 root   root           6 8月  12 2015 src
drwxr-xr-x. 12 songyu songyu      4096 11月 19 2018 zookeeper-3.4.12
-rw-r--r--.  1 root   root      161083 11月 30 19:14 zookeeper.out
```

* 将`elasticsearch-7.5.1-linux-x86_64.tar.gz`包解压至`/usr/local/`路径：

```
[root@localhost local]# tar -zxvf elasticsearch-7.5.1-linux-x86_64.tar.gz 
elasticsearch-7.5.1/
elasticsearch-7.5.1/lib/
elasticsearch-7.5.1/lib/elasticsearch-7.5.1.jar
elasticsearch-7.5.1/lib/elasticsearch-x-content-7.5.1.jar
elasticsearch-7.5.1/lib/elasticsearch-cli-7.5.1.jar
elasticsearch-7.5.1/lib/elasticsearch-core-7.5.1.jar
elasticsearch-7.5.1/lib/elasticsearch-secure-sm-7.5.1.jar
elasticsearch-7.5.1/lib/elasticsearch-geo-7.5.1.jar
elasticsearch-7.5.1/lib/lucene-core-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-analyzers-common-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-backward-codecs-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-grouping-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-highlighter-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-join-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-memory-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-misc-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-queries-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-queryparser-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-sandbox-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-spatial-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-spatial-extras-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-spatial3d-8.3.0.jar
elasticsearch-7.5.1/lib/lucene-suggest-8.3.0.jar
elasticsearch-7.5.1/lib/hppc-0.8.1.jar
elasticsearch-7.5.1/lib/joda-time-2.10.3.jar
...
```

* 修改文件拥有者：

```
[root@localhost local]# chown -R songyu ./elasticsearch-7.5.1
[root@localhost local]# ll
总用量 283464
drwxr-xr-x.  2 root   root           6 8月  12 2015 bin
drwxr-xr-x.  9 songyu root        4096 12月 17 07:01 elasticsearch-7.5.1
-rw-r--r--.  1 root   root   290094012 1月   7 14:43 elasticsearch-7.5.1-linux-x86_64.tar.gz
drwxr-xr-x.  2 root   root           6 8月  12 2015 etc
drwxr-xr-x.  2 root   root           6 8月  12 2015 games
drwxr-xr-x.  2 root   root           6 8月  12 2015 include
drwxr-xr-x.  7 root   root          94 11月 19 2018 kafka_2.12-2.0.0
drwxr-xr-x.  2 root   root           6 8月  12 2015 lib
drwxr-xr-x.  2 root   root           6 8月  12 2015 lib64
drwxr-xr-x.  2 root   root           6 8月  12 2015 libexec
drwxr-xr-x.  2 root   root           6 8月  12 2015 sbin
drwxr-xr-x.  5 root   root          46 11月 15 2018 share
drwxr-xr-x.  2 root   root           6 8月  12 2015 src
drwxr-xr-x. 12 songyu songyu      4096 11月 19 2018 zookeeper-3.4.12
-rw-r--r--.  1 root   root      161083 11月 30 19:14 zookeeper.out
```

* 修改`/config/elasticsearch.yml`配置文件，如下所示：

```
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
#cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
 node.name: node-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
#
# Path to log files:
#
#path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
 network.host: 0.0.0.0
#
# Set a custom port for HTTP:
#
 http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
#cluster.initial_master_nodes: ["node-1", "node-2"]
 cluster.initial_master_nodes: ["node-1"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
```

## 启动Elasticsearch

### root用户启动报错

执行以下命令进行启动`./bin/elasticsearch`：

```
[root@localhost elasticsearch-7.5.1]# ./bin/elasticsearch
future versions of Elasticsearch will require Java 11; your Java version from [/usr/java/jdk1.8.0_191-amd64/jre] does not meet this requirement
[2020-01-07T16:16:42,662][WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [localhost.localdomain] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:163) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:125) ~[elasticsearch-cli-7.5.1.jar:7.5.1]
	at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92) ~[elasticsearch-7.5.1.jar:7.5.1]
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:105) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:172) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:349) ~[elasticsearch-7.5.1.jar:7.5.1]
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159) ~[elasticsearch-7.5.1.jar:7.5.1]
	... 6 more
```

上述信息提示不能以`root`用户启动`Elasticsearch`。

### 其它用户启动

遂切换至`songyu`用户执行`./bin/elasticsearch`命令进行启动，报以下错误:

```
[songyu@localhost elasticsearch-7.5.1]$ ./bin/elasticsearch
future versions of Elasticsearch will require Java 11; your Java version from [/usr/java/jdk1.8.0_191-amd64/jre] does not meet this requirement
[2020-01-08T21:41:18,263][INFO ][o.e.e.NodeEnvironment    ] [localhost.localdomain] using [1] data paths, mounts [[/ (rootfs)]], net usable_space [10.1gb], net total_space [13.7gb], types [rootfs]
[2020-01-08T21:41:18,362][INFO ][o.e.e.NodeEnvironment    ] [localhost.localdomain] heap size [990.7mb], compressed ordinary object pointers [true]
[2020-01-08T21:41:18,604][INFO ][o.e.n.Node               ] [localhost.localdomain] node name [localhost.localdomain], node ID [VpU5ozBkQTexK9eYcCO8Lw], cluster name [elasticsearch]
[2020-01-08T21:41:18,606][INFO ][o.e.n.Node               ] [localhost.localdomain] version[7.5.1], pid[4097], build[default/tar/3ae9ac9a93c95bd0cdc054951cf95d88e1e18d96/2019-12-16T22:57:37.835892Z], OS[Linux/3.10.0-327.el7.x86_64/amd64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_191/25.191-b12]
[2020-01-08T21:41:18,607][INFO ][o.e.n.Node               ] [localhost.localdomain] JVM home [/usr/java/jdk1.8.0_191-amd64/jre]
[2020-01-08T21:41:18,621][INFO ][o.e.n.Node               ] [localhost.localdomain] JVM arguments [-Des.networkaddress.cache.ttl=60, -Des.networkaddress.cache.negative.ttl=10, -XX:+AlwaysPreTouch, -Xss1m, -Djava.awt.headless=true, -Dfile.encoding=UTF-8, -Djna.nosys=true, -XX:-OmitStackTraceInFastThrow, -Dio.netty.noUnsafe=true, -Dio.netty.noKeySetOptimization=true, -Dio.netty.recycler.maxCapacityPerThread=0, -Dio.netty.allocator.numDirectArenas=0, -Dlog4j.shutdownHookEnabled=false, -Dlog4j2.disable.jmx=true, -Djava.locale.providers=COMPAT, -Xms1g, -Xmx1g, -XX:+UseConcMarkSweepGC, -XX:CMSInitiatingOccupancyFraction=75, -XX:+UseCMSInitiatingOccupancyOnly, -Djava.io.tmpdir=/tmp/elasticsearch-4876338172898090526, -XX:+HeapDumpOnOutOfMemoryError, -XX:HeapDumpPath=data, -XX:ErrorFile=logs/hs_err_pid%p.log, -XX:+PrintGCDetails, -XX:+PrintGCDateStamps, -XX:+PrintTenuringDistribution, -XX:+PrintGCApplicationStoppedTime, -Xloggc:logs/gc.log, -XX:+UseGCLogFileRotation, -XX:NumberOfGCLogFiles=32, -XX:GCLogFileSize=64m, -XX:MaxDirectMemorySize=536870912, -Des.path.home=/usr/local/elasticsearch-7.5.1, -Des.path.conf=/usr/local/elasticsearch-7.5.1/config, -Des.distribution.flavor=default, -Des.distribution.type=tar, -Des.bundled_jdk=true]
[2020-01-08T21:42:17,667][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [aggs-matrix-stats]
[2020-01-08T21:42:17,705][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [analysis-common]
[2020-01-08T21:42:17,706][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [flattened]
[2020-01-08T21:42:17,706][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [frozen-indices]
[2020-01-08T21:42:17,707][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [ingest-common]
[2020-01-08T21:42:17,708][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [ingest-geoip]
[2020-01-08T21:42:17,709][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [ingest-user-agent]
[2020-01-08T21:42:17,710][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [lang-expression]
[2020-01-08T21:42:17,711][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [lang-mustache]
[2020-01-08T21:42:17,712][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [lang-painless]
[2020-01-08T21:42:17,713][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [mapper-extras]
[2020-01-08T21:42:17,714][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [parent-join]
[2020-01-08T21:42:17,715][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [percolator]
[2020-01-08T21:42:17,715][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [rank-eval]
[2020-01-08T21:42:17,716][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [reindex]
[2020-01-08T21:42:17,717][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [repository-url]
[2020-01-08T21:42:17,717][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [search-business-rules]
[2020-01-08T21:42:17,719][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [spatial]
[2020-01-08T21:42:17,720][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [transform]
[2020-01-08T21:42:17,721][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [transport-netty4]
[2020-01-08T21:42:17,722][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [vectors]
[2020-01-08T21:42:17,722][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-analytics]
[2020-01-08T21:42:17,723][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-ccr]
[2020-01-08T21:42:17,724][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-core]
[2020-01-08T21:42:17,724][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-deprecation]
[2020-01-08T21:42:17,725][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-enrich]
[2020-01-08T21:42:17,726][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-graph]
[2020-01-08T21:42:17,726][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-ilm]
[2020-01-08T21:42:17,727][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-logstash]
[2020-01-08T21:42:17,728][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-ml]
[2020-01-08T21:42:17,728][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-monitoring]
[2020-01-08T21:42:17,730][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-rollup]
[2020-01-08T21:42:17,731][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-security]
[2020-01-08T21:42:17,732][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-sql]
[2020-01-08T21:42:17,733][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-voting-only-node]
[2020-01-08T21:42:17,734][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-watcher]
[2020-01-08T21:42:17,737][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] no plugins loaded
[2020-01-08T21:42:42,307][INFO ][o.e.x.s.a.s.FileRolesStore] [localhost.localdomain] parsed [0] roles from file [/usr/local/elasticsearch-7.5.1/config/roles.yml]
[2020-01-08T21:42:47,272][INFO ][o.e.x.m.p.l.CppLogMessageHandler] [localhost.localdomain] [controller/4276] [Main.cc@110] controller (64 bit): Version 7.5.1 (Build ae3c3c51b849be) Copyright (c) 2019 Elasticsearch BV
[2020-01-08T21:42:49,522][DEBUG][o.e.a.ActionModule       ] [localhost.localdomain] Using REST wrapper from plugin org.elasticsearch.xpack.security.Security
[2020-01-08T21:42:50,598][INFO ][o.e.d.DiscoveryModule    ] [localhost.localdomain] using discovery type [zen] and seed hosts providers [settings]
[2020-01-08T21:42:55,717][INFO ][o.e.n.Node               ] [localhost.localdomain] initialized
[2020-01-08T21:42:55,726][INFO ][o.e.n.Node               ] [localhost.localdomain] starting ...
[2020-01-08T21:42:57,487][INFO ][o.e.t.TransportService   ] [localhost.localdomain] publish_address {192.168.80.130:9300}, bound_addresses {[::]:9300}
[2020-01-08T21:42:58,914][INFO ][o.e.b.BootstrapChecks    ] [localhost.localdomain] bound or publishing to a non-loopback address, enforcing bootstrap checks
ERROR: [4] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2]: max number of threads [3821] for user [songyu] is too low, increase to at least [4096]
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[4]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
[2020-01-08T21:42:59,042][INFO ][o.e.n.Node               ] [localhost.localdomain] stopping ...
[2020-01-08T21:42:59,544][INFO ][o.e.n.Node               ] [localhost.localdomain] stopped
[2020-01-08T21:42:59,545][INFO ][o.e.n.Node               ] [localhost.localdomain] closing ...
[2020-01-08T21:42:59,649][INFO ][o.e.n.Node               ] [localhost.localdomain] closed
[2020-01-08T21:42:59,654][INFO ][o.e.x.m.p.NativeController] [localhost.localdomain] Native controller process has stopped - no new native processes can be started
```

* 错误(1) 当前用户的软、硬限制过小，即es能打开的最大文件数量过小。在设定上，通常软限制会比硬限制小，硬限制是严格的设定，必定不能超过这个数值；软限制是警告的设定，可以超过这个设定的值，若超过则会有警告信息。

```
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
```

查看当前用户的软限制`ulimit -S -n`、硬限制`ulimit -H -n`：

```
[songyu@localhost elasticsearch-7.5.1]$ ulimit -S -n
1024
[songyu@localhost elasticsearch-7.5.1]$ ulimit -H -n
4096
```

修改软、硬限制数量，在`/etc/security/limits.conf`文件中追加以下配置：

```
* soft nofile 65535
* hard nofile 65537
```

* 错误(2) 当前用户的最大线程数过小。

```
[2]: max number of threads [3821] for user [songyu] is too low, increase to at least [4096]
```

在`/etc/security/limits.conf`文件中追加以下配置：

```
* soft nproc 4096
* hard nproc 4096
```

* 错误(3)

```
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

在`/etc/sysctl.conf`文件中追加以下配置：

```
vm.max_map_count=262144
```

执行命令`sysctl -p`使配置生效。

* 错误(4)

```
[4]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
```

将`/config/elasticsearch.yml`文件中`#node.name: node-1`、`#cluster.initial_master_nodes: ["node-1", "node-2"]`前面的注释去掉，并且后者仅保留一个节点即可，注意格式。

```
 node.name: node-1
 cluster.initial_master_nodes: ["node-1"]
```

### 验证

`ElasticSearch`的默认HTTP端口是`9200`，TCP端口是`9300`，执行以下命令访问`9200`端口：

```
[songyu@localhost ~]$ curl localhost:9200
{
  "name" : "node-1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "lWa_EOeXR-uPMKrfoGqGRA",
  "version" : {
    "number" : "7.5.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "3ae9ac9a93c95bd0cdc054951cf95d88e1e18d96",
    "build_date" : "2019-12-16T22:57:37.835892Z",
    "build_snapshot" : false,
    "lucene_version" : "8.3.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

或者在浏览器中直接访问`9200`端口，访问结果和命令行访问一样，都是一个JSON对象。

### 后台启动

```
[songyu@localhost elasticsearch-7.5.1]$ ./bin/elasticsearch -d
```

## 安装Kibana

`Kibana`的安装步骤如下：
* 进入[Kibana官方网站](https://www.elastic.co/cn/downloads/kibana)，此处选择当前的版本`7.5.1版本`，找到`LINUX`，点击即可下载
* 使用[Bitvise SSH Client](https://www.bitvise.com/ssh-client)工具将下载的`kibana-7.5.1-linux-x86_64.tar.gz`包复制到`/usr/local/`路径，复制完毕后查看该路径下是否有文件：

```
[root@localhost local]# ll
总用量 233060
drwxr-xr-x.  2 root   root           6 8月  12 2015 bin
drwxr-xr-x. 10 songyu root        4096 1月   8 11:15 elasticsearch-7.5.1
drwxr-xr-x.  2 root   root           6 8月  12 2015 etc
drwxr-xr-x.  2 root   root           6 8月  12 2015 games
drwxr-xr-x.  2 root   root           6 8月  12 2015 include
drwxr-xr-x.  7 root   root          94 11月 19 2018 kafka_2.12-2.0.0
-rw-r--r--.  1 root   root   238481011 1月  13 21:42 kibana-7.5.1-linux-x86_64.tar.gz
drwxr-xr-x.  2 root   root           6 8月  12 2015 lib
drwxr-xr-x.  2 root   root           6 8月  12 2015 lib64
drwxr-xr-x.  2 root   root           6 8月  12 2015 libexec
drwxr-xr-x.  2 root   root           6 8月  12 2015 sbin
drwxr-xr-x.  5 root   root          46 11月 15 2018 share
drwxr-xr-x.  2 root   root           6 8月  12 2015 src
drwxr-xr-x. 12 songyu songyu      4096 11月 19 2018 zookeeper-3.4.12
-rw-r--r--.  1 root   root      161083 11月 30 19:14 zookeeper.out
```

* 将`kibana-7.5.1-linux-x86_64.tar.gz`包解压至`/usr/local/`路径：

```
[root@localhost local]# tar -zxvf kibana-7.5.1-linux-x86_64.tar.gz
```

* 修改文件拥有者：

```
[root@localhost local]# chown -R songyu ./kibana-7.5.1-linux-x86_64
[root@localhost local]# ll
总用量 233064
drwxr-xr-x.  2 root   root           6 8月  12 2015 bin
drwxr-xr-x. 10 songyu root        4096 1月   8 11:15 elasticsearch-7.5.1
drwxr-xr-x.  2 root   root           6 8月  12 2015 etc
drwxr-xr-x.  2 root   root           6 8月  12 2015 games
drwxr-xr-x.  2 root   root           6 8月  12 2015 include
drwxr-xr-x.  7 root   root          94 11月 19 2018 kafka_2.12-2.0.0
drwxr-xr-x. 13 songyu root        4096 1月  13 22:03 kibana-7.5.1-linux-x86_64
-rw-r--r--.  1 root   root   238481011 1月  13 21:42 kibana-7.5.1-linux-x86_64.tar.gz
drwxr-xr-x.  2 root   root           6 8月  12 2015 lib
drwxr-xr-x.  2 root   root           6 8月  12 2015 lib64
drwxr-xr-x.  2 root   root           6 8月  12 2015 libexec
drwxr-xr-x.  2 root   root           6 8月  12 2015 sbin
drwxr-xr-x.  5 root   root          46 11月 15 2018 share
drwxr-xr-x.  2 root   root           6 8月  12 2015 src
drwxr-xr-x. 12 songyu songyu      4096 11月 19 2018 zookeeper-3.4.12
-rw-r--r--.  1 root   root      161083 11月 30 19:14 zookeeper.out
```

* 修改`/config/kibana.yml`配置文件中的`server.host`、`elasticsearch.hosts`，如下所示：

```
# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
#server.host: "localhost"
 server.host: "192.168.80.130"
```

```
# The URLs of the Elasticsearch instances to use for all your queries.
#elasticsearch.hosts: ["http://localhost:9200"]
 elasticsearch.hosts: "http://192.168.80.130:9200"
```

### 启动

执行`./bin/kibana`命令进行启动，报以下错误：

```
[songyu@localhost kibana-7.5.1-linux-x86_64]$ ./bin/kibana
  log   [14:33:34.730] [info][plugins-system] Setting up [15] plugins: [code,security,licensing,timelion,features,spaces,translations,eui_utils,uiActions,newsfeed,expressions,data,inspector,embeddable,advancedUiActions]
  log   [14:33:34.740] [info][code][plugins] Setting up plugin
  log   [14:33:34.743] [info][plugins][security] Setting up plugin
  log   [14:33:34.745] [warning][config][plugins][security] Generating a random key for xpack.security.encryptionKey. To prevent sessions from being invalidated on restart, please set xpack.security.encryptionKey in kibana.yml
  log   [14:33:34.878] [warning][config][plugins][security] Session cookies will be transmitted over insecure connections. This is not recommended.
  log   [14:33:35.008] [info][licensing][plugins] Setting up plugin
  log   [14:33:35.069] [info][plugins][timelion] Setting up plugin
  log   [14:33:35.074] [info][features][plugins] Setting up plugin
  log   [14:33:35.076] [info][plugins][spaces] Setting up plugin
  log   [14:33:35.093] [info][plugins][translations] Setting up plugin
  log   [14:33:35.095] [info][data][plugins] Setting up plugin
  log   [14:33:35.443] [error][data][elasticsearch] Request error, retrying
GET http://192.168.80.130:9200/_xpack => connect ECONNREFUSED 192.168.80.130:9200
  log   [14:34:29.929] [warning][data][elasticsearch] Unable to revive connection: http://192.168.80.130:9200/
  log   [14:34:29.976] [warning][data][elasticsearch] No living connections
  log   [14:34:29.979] [warning][licensing][plugins] License information could not be obtained from Elasticsearch for the [data] cluster. Error: No Living connections
  log   [14:34:37.328] [warning][legacy-plugins] Skipping non-plugin directory at /usr/local/kibana-7.5.1-linux-x86_64/src/legacy/core_plugins/visualizations
  log   [14:34:43.489] [info][plugins-system] Starting [8] plugins: [code,security,licensing,timelion,features,spaces,translations,data]
  log   [14:34:45.334] [info][licensing][plugins] Imported changed license information from Elasticsearch for the [data] cluster: type: basic | status: active
  log   [14:34:45.422] [info][migrations] Creating index .kibana_task_manager_1.
  log   [14:34:45.917] [info][migrations] Creating index .kibana_1.
  log   [14:35:16.258] [warning][migrations] Unable to connect to Elasticsearch. Error: Request Timeout after 30000ms
  log   [14:35:45.530] [warning][migrations] Unable to connect to Elasticsearch. Error: [resource_already_exists_exception] index [.kibana_1/SJxIdZIUQSyfBtQz57lM5w] already exists, with { index_uuid="SJxIdZIUQSyfBtQz57lM5w" & index=".kibana_1" }
  log   [14:35:46.449] [warning][migrations] Another Kibana instance appears to be migrating the index. Waiting for that migration to complete. If no other Kibana instance is attempting migrations, you can get past this message by deleting index .kibana_1 and restarting Kibana.
  log   [14:35:52.973] [warning][migrations] Unable to connect to Elasticsearch. Error: [resource_already_exists_exception] index [.kibana_task_manager_1/aVhOfsd7Re21RbxZX78i5w] already exists, with { index_uuid="aVhOfsd7Re21RbxZX78i5w" & index=".kibana_task_manager_1" }
  log   [14:35:53.379] [warning][migrations] Another Kibana instance appears to be migrating the index. Waiting for that migration to complete. If no other Kibana instance is attempting migrations, you can get past this message by deleting index .kibana_task_manager_1 and restarting Kibana.
  log   [14:37:03.576] [warning][licensing][plugins] License information could not be obtained from Elasticsearch for the [data] cluster. Error: Request Timeout after 30000ms
...
```

### 后台启动

执行`./bin/kibana &`命令进行后台启动

* [参考：Node](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/modules-node.html#modules-node)