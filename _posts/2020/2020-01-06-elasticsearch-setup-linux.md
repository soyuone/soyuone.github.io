---
layout: post
title: "Elasticsearch在Linux平台的安装和配置"
categories: 大数据
tags: 大数据 Elasticsearch
author: Kopite
---

* content
{:toc}


本文介绍`Elasticsearch`在Linux平台的安装及配置，`Elasticsearch`的安装需要Java的支持，与Java版本对应关系见[Elasticsearch and JVM](https://www.elastic.co/cn/support/matrix#matrix_jvm)。



## 安装Elasticsearch

`Elasticsearch`的安装步骤如下：
* 进入[Elasticsearch官方网站](https://www.elastic.co/cn/downloads/elasticsearch)，此处选择当前的版本`7.5.1版本`，找到`LINUX`，点击即可下载
* 使用[Bitvise SSH Client](https://www.bitvise.com/ssh-client)工具将下载的`elasticsearch-7.5.1-linux-x86_64.tar.gz`包复制到`/usr/local/`路径，复制完毕后查看该路径下是否有文件：

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

## 启动Elasticsearch

### root启动报错

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

遂切换至`songyu`用户执行`./bin/elasticsearch`命令进行启动，启动成功:

```
[songyu@localhost elasticsearch-7.5.1]$ ./bin/elasticsearch
future versions of Elasticsearch will require Java 11; your Java version from [/usr/java/jdk1.8.0_191-amd64/jre] does not meet this requirement
[2020-01-08T13:16:24,718][INFO ][o.e.e.NodeEnvironment    ] [localhost.localdomain] using [1] data paths, mounts [[/ (rootfs)]], net usable_space [10.1gb], net total_space [13.7gb], types [rootfs]
[2020-01-08T13:16:24,778][INFO ][o.e.e.NodeEnvironment    ] [localhost.localdomain] heap size [990.7mb], compressed ordinary object pointers [true]
[2020-01-08T13:16:24,832][INFO ][o.e.n.Node               ] [localhost.localdomain] node name [localhost.localdomain], node ID [VpU5ozBkQTexK9eYcCO8Lw], cluster name [elasticsearch]
[2020-01-08T13:16:24,833][INFO ][o.e.n.Node               ] [localhost.localdomain] version[7.5.1], pid[9876], build[default/tar/3ae9ac9a93c95bd0cdc054951cf95d88e1e18d96/2019-12-16T22:57:37.835892Z], OS[Linux/3.10.0-327.el7.x86_64/amd64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_191/25.191-b12]
[2020-01-08T13:16:24,835][INFO ][o.e.n.Node               ] [localhost.localdomain] JVM home [/usr/java/jdk1.8.0_191-amd64/jre]
[2020-01-08T13:16:24,837][INFO ][o.e.n.Node               ] [localhost.localdomain] JVM arguments [-Des.networkaddress.cache.ttl=60, -Des.networkaddress.cache.negative.ttl=10, -XX:+AlwaysPreTouch, -Xss1m, -Djava.awt.headless=true, -Dfile.encoding=UTF-8, -Djna.nosys=true, -XX:-OmitStackTraceInFastThrow, -Dio.netty.noUnsafe=true, -Dio.netty.noKeySetOptimization=true, -Dio.netty.recycler.maxCapacityPerThread=0, -Dio.netty.allocator.numDirectArenas=0, -Dlog4j.shutdownHookEnabled=false, -Dlog4j2.disable.jmx=true, -Djava.locale.providers=COMPAT, -Xms1g, -Xmx1g, -XX:+UseConcMarkSweepGC, -XX:CMSInitiatingOccupancyFraction=75, -XX:+UseCMSInitiatingOccupancyOnly, -Djava.io.tmpdir=/tmp/elasticsearch-5143197216165547241, -XX:+HeapDumpOnOutOfMemoryError, -XX:HeapDumpPath=data, -XX:ErrorFile=logs/hs_err_pid%p.log, -XX:+PrintGCDetails, -XX:+PrintGCDateStamps, -XX:+PrintTenuringDistribution, -XX:+PrintGCApplicationStoppedTime, -Xloggc:logs/gc.log, -XX:+UseGCLogFileRotation, -XX:NumberOfGCLogFiles=32, -XX:GCLogFileSize=64m, -XX:MaxDirectMemorySize=536870912, -Des.path.home=/usr/local/elasticsearch-7.5.1, -Des.path.conf=/usr/local/elasticsearch-7.5.1/config, -Des.distribution.flavor=default, -Des.distribution.type=tar, -Des.bundled_jdk=true]
[2020-01-08T13:17:07,508][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [aggs-matrix-stats]
[2020-01-08T13:17:07,551][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [analysis-common]
[2020-01-08T13:17:07,552][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [flattened]
[2020-01-08T13:17:07,553][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [frozen-indices]
[2020-01-08T13:17:07,554][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [ingest-common]
[2020-01-08T13:17:07,554][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [ingest-geoip]
[2020-01-08T13:17:07,555][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [ingest-user-agent]
[2020-01-08T13:17:07,556][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [lang-expression]
[2020-01-08T13:17:07,556][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [lang-mustache]
[2020-01-08T13:17:07,558][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [lang-painless]
[2020-01-08T13:17:07,559][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [mapper-extras]
[2020-01-08T13:17:07,560][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [parent-join]
[2020-01-08T13:17:07,560][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [percolator]
[2020-01-08T13:17:07,561][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [rank-eval]
[2020-01-08T13:17:07,562][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [reindex]
[2020-01-08T13:17:07,563][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [repository-url]
[2020-01-08T13:17:07,564][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [search-business-rules]
[2020-01-08T13:17:07,564][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [spatial]
[2020-01-08T13:17:07,565][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [transform]
[2020-01-08T13:17:07,566][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [transport-netty4]
[2020-01-08T13:17:07,566][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [vectors]
[2020-01-08T13:17:07,567][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-analytics]
[2020-01-08T13:17:07,568][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-ccr]
[2020-01-08T13:17:07,569][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-core]
[2020-01-08T13:17:07,570][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-deprecation]
[2020-01-08T13:17:07,571][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-enrich]
[2020-01-08T13:17:07,571][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-graph]
[2020-01-08T13:17:07,572][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-ilm]
[2020-01-08T13:17:07,573][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-logstash]
[2020-01-08T13:17:07,573][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-ml]
[2020-01-08T13:17:07,574][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-monitoring]
[2020-01-08T13:17:07,575][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-rollup]
[2020-01-08T13:17:07,576][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-security]
[2020-01-08T13:17:07,577][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-sql]
[2020-01-08T13:17:07,577][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-voting-only-node]
[2020-01-08T13:17:07,578][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] loaded module [x-pack-watcher]
[2020-01-08T13:17:07,580][INFO ][o.e.p.PluginsService     ] [localhost.localdomain] no plugins loaded
[2020-01-08T13:17:25,461][INFO ][o.e.x.s.a.s.FileRolesStore] [localhost.localdomain] parsed [0] roles from file [/usr/local/elasticsearch-7.5.1/config/roles.yml]
[2020-01-08T13:17:32,384][INFO ][o.e.x.m.p.l.CppLogMessageHandler] [localhost.localdomain] [controller/9965] [Main.cc@110] controller (64 bit): Version 7.5.1 (Build ae3c3c51b849be) Copyright (c) 2019 Elasticsearch BV
[2020-01-08T13:17:34,573][DEBUG][o.e.a.ActionModule       ] [localhost.localdomain] Using REST wrapper from plugin org.elasticsearch.xpack.security.Security
[2020-01-08T13:17:35,706][INFO ][o.e.d.DiscoveryModule    ] [localhost.localdomain] using discovery type [zen] and seed hosts providers [settings]
[2020-01-08T13:17:40,723][INFO ][o.e.n.Node               ] [localhost.localdomain] initialized
[2020-01-08T13:17:40,723][INFO ][o.e.n.Node               ] [localhost.localdomain] starting ...
[2020-01-08T13:17:42,730][INFO ][o.e.t.TransportService   ] [localhost.localdomain] publish_address {127.0.0.1:9300}, bound_addresses {[::1]:9300}, {127.0.0.1:9300}
[2020-01-08T13:17:44,519][WARN ][o.e.b.BootstrapChecks    ] [localhost.localdomain] max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2020-01-08T13:17:44,546][WARN ][o.e.b.BootstrapChecks    ] [localhost.localdomain] max number of threads [3821] for user [songyu] is too low, increase to at least [4096]
[2020-01-08T13:17:44,546][WARN ][o.e.b.BootstrapChecks    ] [localhost.localdomain] max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[2020-01-08T13:17:44,547][WARN ][o.e.b.BootstrapChecks    ] [localhost.localdomain] the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
[2020-01-08T13:17:44,549][INFO ][o.e.c.c.Coordinator      ] [localhost.localdomain] cluster UUID [lWa_EOeXR-uPMKrfoGqGRA]
[2020-01-08T13:17:45,033][INFO ][o.e.c.c.ClusterBootstrapService] [localhost.localdomain] no discovery configuration found, will perform best-effort cluster bootstrapping after [3s] unless existing master is discovered
[2020-01-08T13:17:46,855][INFO ][o.e.c.s.MasterService    ] [localhost.localdomain] elected-as-master ([1] nodes joined)[{localhost.localdomain}{VpU5ozBkQTexK9eYcCO8Lw}{rmmyISXPSFG0iOU486TQ7g}{127.0.0.1}{127.0.0.1:9300}{dilm}{ml.machine_memory=1033527296, xpack.installed=true, ml.max_open_jobs=20} elect leader, _BECOME_MASTER_TASK_, _FINISH_ELECTION_], term: 2, version: 18, delta: master node changed {previous [], current [{localhost.localdomain}{VpU5ozBkQTexK9eYcCO8Lw}{rmmyISXPSFG0iOU486TQ7g}{127.0.0.1}{127.0.0.1:9300}{dilm}{ml.machine_memory=1033527296, xpack.installed=true, ml.max_open_jobs=20}]}
[2020-01-08T13:17:47,514][INFO ][o.e.c.s.ClusterApplierService] [localhost.localdomain] master node changed {previous [], current [{localhost.localdomain}{VpU5ozBkQTexK9eYcCO8Lw}{rmmyISXPSFG0iOU486TQ7g}{127.0.0.1}{127.0.0.1:9300}{dilm}{ml.machine_memory=1033527296, xpack.installed=true, ml.max_open_jobs=20}]}, term: 2, version: 18, reason: Publication{term=2, version=18}
[2020-01-08T13:17:49,172][INFO ][o.e.h.AbstractHttpServerTransport] [localhost.localdomain] publish_address {127.0.0.1:9200}, bound_addresses {[::1]:9200}, {127.0.0.1:9200}
[2020-01-08T13:17:49,173][INFO ][o.e.n.Node               ] [localhost.localdomain] started
[2020-01-08T13:17:49,885][INFO ][o.e.l.LicenseService     ] [localhost.localdomain] license [2e8d3241-b7be-49f9-b4bb-6d6fcca0bae5] mode [basic] - valid
[2020-01-08T13:17:49,889][INFO ][o.e.x.s.s.SecurityStatusChangeListener] [localhost.localdomain] Active license is now [BASIC]; Security is disabled
[2020-01-08T13:17:49,976][INFO ][o.e.g.GatewayService     ] [localhost.localdomain] recovered [0] indices into cluster_state
...
```

### 验证

`ElasticSearch`的默认HTTP端口是`9200`，TCP端口是`9300`，执行以下命令访问`9200`端口：

```
[songyu@localhost config]$ curl localhost:9200
{
  "name" : "localhost.localdomain",
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

* [参考：ElasticSearch 学习笔记](https://segmentfault.com/a/1190000016651566)
* [参考：Elasticsearch 7.x 最详细安装及配置](https://segmentfault.com/a/1190000020134018)
* [参考：elasticsearch的安装及常见问题解决](https://blog.csdn.net/dajienet/article/details/80009391)