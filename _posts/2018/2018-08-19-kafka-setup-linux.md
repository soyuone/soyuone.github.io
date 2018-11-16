---
layout: post
title: "Zookeeper、Kafka在Linux平台的安装和配置"
categories: 大数据
tags: 大数据 apache Zookeeper Kafka MQ Linux
author: Kopite
---

* content
{:toc}


`Kafka`是使用`Java`开发的应用程序，所以它可以运行在`windows`、`MacOS`和`Linux`等多种操作系统上。
<br>
<br>
运行`Zookeeper`和`Kafka`需要`Java`运行时版本，所以在安装`Zookeeper`和`Kafka`之前，需要先安装`Java`环境。



## 安装Zookeeper

![](/image/2018/2018-08-19-kafka-setup-linux-1.PNG)

`Kafka`使用`Zookeeper`保存集群的元数据信息和消费者信息，见上图所示，`Zookeeper`的安装步骤如下（`Kafka`发行版自带了`Zookeeper`，可直接从脚本启动）：
* 进入[Apache Zookeeper官方网站](http://zookeeper.apache.org/)，选择合适的镜像站点及Apache Zookeeper版本进行下载，此处使用[www-eu.apache.org/dist/zookeeper/](http://www-eu.apache.org/dist/zookeeper/)镜像站点进行下载，选择当前的稳定版`3.4.12版本`，找到`zookeeper-3.4.12.tar.gz`，点击即可下载
* 使用[Bitvise SSH Client](https://www.bitvise.com/ssh-client)工具将下载的`zookeeper-3.4.12.tar.gz`包复制到`/usr/local/`路径，复制完毕后查看该路径下是否有文件：

```
[root@ local]# ls -l
总用量 35816
drwxr-xr-x. 9 root root     4096 6月  30 2017 apache-tomcat-7.0.78
drwxr-xr-x. 2 root root        6 8月  12 2015 bin
drwxr-xr-x. 2 root root        6 8月  12 2015 etc
drwxr-xr-x. 2 root root        6 8月  12 2015 games
drwxr-xr-x. 2 root root        6 8月  12 2015 include
drwxr-xr-x. 2 root root        6 8月  12 2015 lib
drwxr-xr-x. 2 root root        6 8月  12 2015 lib64
drwxr-xr-x. 2 root root        6 8月  12 2015 libexec
drwxr-xr-x. 2 root root        6 8月  12 2015 sbin
drwxr-xr-x. 5 root root       46 6月  20 2017 share
drwxr-xr-x. 2 root root        6 8月  12 2015 src
-rw-r--r--. 1 root root 36667596 8月  19 16:03 zookeeper-3.4.12.tar.gz
```

* 将`zookeeper-3.4.12.tar.gz`包解压至`/usr/local/`路径：

```
[root@ local]# tar -zxvf zookeeper-3.4.12.tar.gz
zookeeper-3.4.12/
zookeeper-3.4.12/contrib/
zookeeper-3.4.12/contrib/rest/
zookeeper-3.4.12/contrib/rest/lib/
zookeeper-3.4.12/contrib/rest/lib/slf4j-log4j12-1.6.1.jar
zookeeper-3.4.12/contrib/rest/lib/jackson-core-asl-1.1.1.jar
zookeeper-3.4.12/contrib/rest/lib/jsr311-api-1.1.1.jar
zookeeper-3.4.12/contrib/rest/lib/jettison-1.1.jar
zookeeper-3.4.12/contrib/rest/lib/log4j-1.2.15.jar
zookeeper-3.4.12/contrib/rest/lib/activation-1.1.jar
zookeeper-3.4.12/contrib/rest/lib/jersey-server-1.1.5.1.jar
zookeeper-3.4.12/contrib/rest/lib/jaxb-impl-2.1.12.jar
zookeeper-3.4.12/contrib/rest/lib/jersey-core-1.1.5.1.jar
zookeeper-3.4.12/contrib/rest/lib/grizzly-http-servlet-1.9.8.jar
zookeeper-3.4.12/contrib/rest/lib/grizzly-utils-1.9.8.jar
zookeeper-3.4.12/contrib/rest/lib/servlet-api-2.5.jar
zookeeper-3.4.12/contrib/rest/lib/grizzly-http-1.9.8.jar
zookeeper-3.4.12/contrib/rest/lib/grizzly-portunif-1.9.8.jar
zookeeper-3.4.12/contrib/rest/lib/stax-api-1.0.1.jar
zookeeper-3.4.12/contrib/rest/lib/asm-3.1.jar
zookeeper-3.4.12/contrib/rest/lib/grizzly-rcm-1.9.8.jar
zookeeper-3.4.12/contrib/rest/lib/grizzly-servlet-webserver-1.9.8.jar
zookeeper-3.4.12/contrib/rest/lib/grizzly-framework-1.9.8.jar
zookeeper-3.4.12/contrib/rest/lib/jersey-json-1.1.5.1.jar
...
```

* 进入`/usr/local/zookeeper-3.4.12/conf`目录，复制文件`zoo_sample.cfg`并命名为`zoo.cfg`：

```
[root@ conf]# cp zoo_sample.cfg zoo.cfg
```

* 打开`zoo.cfg`文件，创建`/zookeeper-3.4.12/data`目录，修改数据目录为`dataDir=/usr/local/zookeeper-3.4.12/data`，保存并关闭：

```
# number of milliseconds of each tick
# Zookeeper定义的基准时间间隔，单位：毫秒
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
# 用于在从节点与主节点之间建立初始化连接的时间上限
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
# 允许从节点与主节点处于不同步状态的时间上限
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
# 数据目录
dataDir=/usr/local/zookeeper-3.4.12/data
# the port at which the clients will connect
# 客户端访问Zookeeper的端口号
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

* 在`/etc/profile`尾部追加如下内容：

```
# zookeeper-3.4.12 config start
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.12
export PATH=$ZOOKEEPER_HOME/bin:$PATH
# zookeeper-3.4.12 config end
```

* 使`/etc/profile`文件立即生效：

```
[root@ etc]# source /etc/profile
```

### 启动Zookeeper

输入`zkServer.sh start`，启动`Zookeeper`服务，如打印如下信息则表示启动成功：

```
[root@ zookeeper-3.4.12]# zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

### 查询Zookeeper状态

查询`Zookeeper`状态：

```
[root@ zookeeper-3.4.12]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: standalone
```

### 停止Zookeeper

停止`Zookeeper`服务：

```
[root@ zookeeper-3.4.12]# zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```

### 重启Zookeeper

重启`Zookeeper`服务：

```
[root@ zookeeper-3.4.12]# zkServer.sh restart
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

## Zookeeper集群

`Zookeeper`使用一致性协议，所以建议每个节点里应该包含`奇数`个节点（比如3个、5个等），因为只有当集群里的`大多数`节点处于可用状态，`Zookeeper`才能处理外部的请求。
<br>
<br>
假设有一个包含5个节点的集群，如果要对集群做一些包括更换节点在内的配置更改，需要依次重启每一个节点。不建议一个集群包含超过7个节点，节点过多会降低整个集群的性能。
<br>
<br>
集群需要有一些公共配置，需要列出所有服务器，并且每个服务器还要在数据目录中创建一个`myid`文件，用于指明自己的ID。
<br>
<br>
首先在`/conf/zoo.cfg`文件中按照`server.X=hostname:peerPort:leaderPort`格式添加所有`Zookeeper`服务器的地址，如下所示：
* `X`：服务器的ID，它必须是一个整数，不一定要从`0`开始，也不要求连续
* `hostname`：服务器的ip地址
* `peerPort`：用于节点间通信的TCP端口
* `leaderPort`：用于首领选举的TCP端口

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/usr/local/zookeeper-3.4.12/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.0=192.168.80.129:2888:3888
server.1=192.168.80.130:2888:3888
server.2=192.168.80.131:2888:3888
```

客户端只需要通过`clientPort`就能连接到集群，而集群节点间的通信则需要同时用到这3个端口（`clientPort`、`peerPort`、`leaderPort`）。
<br>
<br>
另外，在`dataDir`目录下创建一个名为`/data/myid`的文件，在文件内部输入对应服务器的`X`值，如下所示：

```
[root@ data]# vim myid
```

完成上述步骤并给`2181`、`2888`、`3888`端口开启防火墙后，就可以依次启动`Zookeeper`服务并查看启动状态，`192.168.80.129`、`192.168.80.130`、`192.168.80.131`的启动状态依次如下所示：

```
[root@ /]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: follower
```

```
[root@ /]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: follower
```

```
[root@ /]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: leader
```

## 安装Kafka Broker

配置好`Java`和`Zookeeper`后，接下来就可以安装`Kafka`：
* 进入[Apache Kafka官方网站](http://kafka.apache.org/)，此处选择`Kafka 2.0.0版本，对应的Scala版本是2.12`，找到`kafka_2.12-2.0.0.tgz`，点击下载
* 使用[Bitvise SSH Client](https://www.bitvise.com/ssh-client)工具将下载的`kafka_2.12-2.0.0.tgz`包复制到`/usr/local/`路径，复制完毕后查看该路径下是否有文件：

```
[root@ local]# ls -l
总用量 48256
drwxr-xr-x.  9 root   root       4096 6月  30 2017 apache-tomcat-7.0.78
drwxr-xr-x.  2 root   root          6 8月  12 2015 bin
drwxr-xr-x.  2 root   root          6 8月  12 2015 etc
drwxr-xr-x.  2 root   root          6 8月  12 2015 games
drwxr-xr-x.  2 root   root          6 8月  12 2015 include
-rw-r--r--.  1 root   root   49405896 8月  19 18:31 kafka_2.12-2.0.0.tgz
drwxr-xr-x.  2 root   root          6 8月  12 2015 lib
drwxr-xr-x.  2 root   root          6 8月  12 2015 lib64
drwxr-xr-x.  2 root   root          6 8月  12 2015 libexec
drwxr-xr-x.  2 root   root          6 8月  12 2015 sbin
drwxr-xr-x.  5 root   root         46 6月  20 2017 share
drwxr-xr-x.  2 root   root          6 8月  12 2015 src
drwxr-xr-x. 11 songyu songyu     4096 8月  19 17:39 zookeeper-3.4.12
```

* 将`kafka_2.12-2.0.0.tgz`包解压至`/usr/local/`路径：

```
[root@ local]# tar -zxvf kafka_2.12-2.0.0.tgz 
kafka_2.12-2.0.0/
kafka_2.12-2.0.0/LICENSE
kafka_2.12-2.0.0/NOTICE
kafka_2.12-2.0.0/bin/
kafka_2.12-2.0.0/bin/connect-distributed.sh
kafka_2.12-2.0.0/bin/connect-standalone.sh
kafka_2.12-2.0.0/bin/kafka-acls.sh
kafka_2.12-2.0.0/bin/kafka-broker-api-versions.sh
kafka_2.12-2.0.0/bin/kafka-configs.sh
kafka_2.12-2.0.0/bin/kafka-console-consumer.sh
kafka_2.12-2.0.0/bin/kafka-console-producer.sh
kafka_2.12-2.0.0/bin/kafka-consumer-groups.sh
kafka_2.12-2.0.0/bin/kafka-consumer-perf-test.sh
kafka_2.12-2.0.0/bin/kafka-delegation-tokens.sh
kafka_2.12-2.0.0/bin/kafka-delete-records.sh
kafka_2.12-2.0.0/bin/kafka-dump-log.sh
kafka_2.12-2.0.0/bin/kafka-log-dirs.sh
kafka_2.12-2.0.0/bin/kafka-mirror-maker.sh
...
``` 

* 创建日志目录`/kafka_2.12-2.0.0/logs`：

```
[root@ kafka_2.12-2.0.0]# pwd
/usr/local/kafka_2.12-2.0.0
[root@ kafka_2.12-2.0.0]# mkdir logs
```

* 修改`/kafka_2.12-2.0.0/config/server.properties`文件，保存并关闭：

```
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
# 套接字服务器监听的地址，如果没有配置，就使用java.net.InetAddress.getCanonicalHostName()的返回值。
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://192.168.80.129:9092

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
# 节点的主机名会通知给生产者和消费者，如果没有配置，"listeners"若配置就使用"listeners"的值，
# 否则就使用java.net.InetAddress.getCanonicalHostName()的返回值
advertised.listeners=PLAINTEXT://192.168.80.129:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
# 日志目录
log.dirs=/usr/local/kafka_2.12-2.0.0/logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended for to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
```

注意：在使用`VMware`安装的`CentOS`系统上安装的`kafka`，其配置文件中的ip地址不可以使用`locahost`，对应的端口`9092`也需要开启。

### 启动Kafka

在当前目录下输入`sh ./bin/kafka-server-start.sh ./config/server.properties`，启动`Kafka`，提示信息报错，原因`Java`版本过低：

```
[root@ kafka_2.12-2.0.0]# sh ./bin/kafka-server-start.sh ./config/server.properties 
Exception in thread "main" java.lang.UnsupportedClassVersionError: kafka/Kafka : Unsupported major.minor version 52.0
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:800)
	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
	at java.net.URLClassLoader.defineClass(URLClassLoader.java:449)
	at java.net.URLClassLoader.access$100(URLClassLoader.java:71)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:361)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
	at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:482)
```

* 将`Java`版本更换为`1.8.0_181`，再次启动`Kafka`，启动失败，原因是由于`Zookeeper`未启动：

```
[root@localhost kafka_2.12-2.0.0]# sh ./bin/kafka-server-start.sh ./config/server.properties 
[2018-08-20 00:14:46,483] INFO Registered kafka:type=kafka.Log4jController MBean (kafka.utils.Log4jControllerRegistration$)
[2018-08-20 00:14:47,914] INFO starting (kafka.server.KafkaServer)
[2018-08-20 00:14:47,916] INFO Connecting to zookeeper on localhost:2181 (kafka.server.KafkaServer)
[2018-08-20 00:14:48,035] INFO [ZooKeeperClient] Initializing a new session to localhost:2181. (kafka.zookeeper.ZooKeeperClient)
[2018-08-20 00:14:48,049] INFO Client environment:zookeeper.version=3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 00:39 GMT (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,049] INFO Client environment:host.name=localhost (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,049] INFO Client environment:java.version=1.8.0_181 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,049] INFO Client environment:java.vendor=Oracle Corporation (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,050] INFO Client environment:java.home=/usr/java/jdk1.8.0_181-amd64/jre (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,050] INFO Client environment:java.class.path=.:/usr/java/jdk1.8.0_181-amd64/lib/dt.jar:/usr/java/jdk1.8.0_181-amd64/lib/tools.jar:/usr/java/jdk1.8.0_181-amd64/jre/lib:/usr/local/kafka_2.12-2.0.0/bin/../libs/activation-1.1.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/aopalliance-repackaged-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/argparse4j-0.7.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/audience-annotations-0.5.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/commons-lang3-3.5.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-api-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-basic-auth-extension-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-file-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-json-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-runtime-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-transforms-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/guava-20.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/hk2-api-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/hk2-locator-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/hk2-utils-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-annotations-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-core-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-databind-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-jaxrs-base-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-jaxrs-json-provider-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-module-jaxb-annotations-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javassist-3.22.0-CR2.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.annotation-api-1.2.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.inject-1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.inject-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.servlet-api-3.1.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.ws.rs-api-2.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jaxb-api-2.3.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-client-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-common-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-container-servlet-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-container-servlet-core-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-hk2-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-media-jaxb-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-server-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-client-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-continuation-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-http-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-io-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-security-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-server-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-servlet-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-servlets-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-util-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jopt-simple-5.0.4.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka_2.12-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka_2.12-2.0.0-sources.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-clients-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-log4j-appender-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-streams-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-streams-examples-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-streams-scala_2.12-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-streams-test-utils-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-tools-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/log4j-1.2.17.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/lz4-java-1.4.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/maven-artifact-3.5.3.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/metrics-core-2.2.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/osgi-resource-locator-1.0.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/plexus-utils-3.1.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/reflections-0.9.11.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/rocksdbjni-5.7.3.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/scala-library-2.12.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/scala-logging_2.12-3.9.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/scala-reflect-2.12.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/slf4j-api-1.7.25.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/slf4j-log4j12-1.7.25.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/snappy-java-1.1.7.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/validation-api-1.1.0.Final.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/zkclient-0.10.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/zookeeper-3.4.13.jar (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,050] INFO Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,050] INFO Client environment:java.io.tmpdir=/tmp (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,050] INFO Client environment:java.compiler=<NA> (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,051] INFO Client environment:os.name=Linux (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,051] INFO Client environment:os.arch=amd64 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,051] INFO Client environment:os.version=3.10.0-327.el7.x86_64 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,051] INFO Client environment:user.name=root (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,051] INFO Client environment:user.home=/root (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,051] INFO Client environment:user.dir=/usr/local/kafka_2.12-2.0.0 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,053] INFO Initiating client connection, connectString=localhost:2181 sessionTimeout=6000 watcher=kafka.zookeeper.ZooKeeperClient$ZooKeeperClientWatcher$@76908cc0 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:48,078] INFO [ZooKeeperClient] Waiting until connected. (kafka.zookeeper.ZooKeeperClient)
[2018-08-20 00:14:48,079] INFO Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:48,129] INFO Socket error occurred: localhost/0:0:0:0:0:0:0:1:2181: 拒绝连接 (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:49,235] INFO Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:49,237] INFO Socket error occurred: localhost/0:0:0:0:0:0:0:1:2181: 拒绝连接 (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:50,340] INFO Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:50,342] INFO Socket error occurred: localhost/0:0:0:0:0:0:0:1:2181: 拒绝连接 (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:51,444] INFO Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:51,446] INFO Socket error occurred: localhost/0:0:0:0:0:0:0:1:2181: 拒绝连接 (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:52,549] INFO Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:52,551] INFO Socket error occurred: localhost/0:0:0:0:0:0:0:1:2181: 拒绝连接 (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:53,652] INFO Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:53,654] INFO Socket error occurred: localhost/127.0.0.1:2181: 拒绝连接 (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:54,083] INFO [ZooKeeperClient] Closing. (kafka.zookeeper.ZooKeeperClient)
[2018-08-20 00:14:54,757] INFO Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:54,862] INFO Session: 0x0 closed (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:14:54,862] INFO EventThread shut down for session: 0x0 (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:14:54,867] INFO [ZooKeeperClient] Closed. (kafka.zookeeper.ZooKeeperClient)
[2018-08-20 00:14:54,871] ERROR Fatal error during KafkaServer startup. Prepare to shutdown (kafka.server.KafkaServer)
kafka.zookeeper.ZooKeeperClientTimeoutException: Timed out waiting for connection while in state: CONNECTING
	at kafka.zookeeper.ZooKeeperClient.$anonfun$waitUntilConnected$3(ZooKeeperClient.scala:230)
	at scala.runtime.java8.JFunction0$mcV$sp.apply(JFunction0$mcV$sp.java:12)
	at kafka.utils.CoreUtils$.inLock(CoreUtils.scala:251)
	at kafka.zookeeper.ZooKeeperClient.waitUntilConnected(ZooKeeperClient.scala:226)
	at kafka.zookeeper.ZooKeeperClient.<init>(ZooKeeperClient.scala:95)
	at kafka.zk.KafkaZkClient$.apply(KafkaZkClient.scala:1581)
	at kafka.server.KafkaServer.createZkClient$1(KafkaServer.scala:348)
	at kafka.server.KafkaServer.initZkClient(KafkaServer.scala:372)
	at kafka.server.KafkaServer.startup(KafkaServer.scala:202)
	at kafka.server.KafkaServerStartable.startup(KafkaServerStartable.scala:38)
	at kafka.Kafka$.main(Kafka.scala:75)
	at kafka.Kafka.main(Kafka.scala)
[2018-08-20 00:14:54,875] INFO shutting down (kafka.server.KafkaServer)
[2018-08-20 00:14:54,880] WARN  (kafka.utils.CoreUtils$)
java.lang.NullPointerException
	at kafka.server.KafkaServer.$anonfun$shutdown$6(KafkaServer.scala:579)
	at kafka.utils.CoreUtils$.swallow(CoreUtils.scala:86)
	at kafka.server.KafkaServer.shutdown(KafkaServer.scala:579)
	at kafka.server.KafkaServer.startup(KafkaServer.scala:329)
	at kafka.server.KafkaServerStartable.startup(KafkaServerStartable.scala:38)
	at kafka.Kafka$.main(Kafka.scala:75)
	at kafka.Kafka.main(Kafka.scala)
[2018-08-20 00:14:54,897] INFO shut down completed (kafka.server.KafkaServer)
[2018-08-20 00:14:54,899] ERROR Exiting Kafka. (kafka.server.KafkaServerStartable)
[2018-08-20 00:14:54,907] INFO shutting down (kafka.server.KafkaServer)
```

* 先启动`Zookeeper`，启动`Kafka`成功，提示信息如下：

```
[root@localhost kafka_2.12-2.0.0]# sh ./bin/kafka-server-start.sh ./config/server.properties 
[2018-08-20 00:24:04,072] INFO Registered kafka:type=kafka.Log4jController MBean (kafka.utils.Log4jControllerRegistration$)
[2018-08-20 00:24:05,075] INFO starting (kafka.server.KafkaServer)
[2018-08-20 00:24:05,078] INFO Connecting to zookeeper on localhost:2181 (kafka.server.KafkaServer)
[2018-08-20 00:24:05,122] INFO [ZooKeeperClient] Initializing a new session to localhost:2181. (kafka.zookeeper.ZooKeeperClient)
[2018-08-20 00:24:05,133] INFO Client environment:zookeeper.version=3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 00:39 GMT (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,133] INFO Client environment:host.name=localhost (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,133] INFO Client environment:java.version=1.8.0_181 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,133] INFO Client environment:java.vendor=Oracle Corporation (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,133] INFO Client environment:java.home=/usr/java/jdk1.8.0_181-amd64/jre (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,134] INFO Client environment:java.class.path=.:/usr/java/jdk1.8.0_181-amd64/lib/dt.jar:/usr/java/jdk1.8.0_181-amd64/lib/tools.jar:/usr/java/jdk1.8.0_181-amd64/jre/lib:/usr/local/kafka_2.12-2.0.0/bin/../libs/activation-1.1.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/aopalliance-repackaged-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/argparse4j-0.7.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/audience-annotations-0.5.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/commons-lang3-3.5.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-api-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-basic-auth-extension-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-file-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-json-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-runtime-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/connect-transforms-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/guava-20.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/hk2-api-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/hk2-locator-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/hk2-utils-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-annotations-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-core-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-databind-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-jaxrs-base-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-jaxrs-json-provider-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jackson-module-jaxb-annotations-2.9.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javassist-3.22.0-CR2.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.annotation-api-1.2.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.inject-1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.inject-2.5.0-b42.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.servlet-api-3.1.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/javax.ws.rs-api-2.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jaxb-api-2.3.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-client-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-common-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-container-servlet-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-container-servlet-core-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-hk2-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-media-jaxb-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jersey-server-2.27.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-client-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-continuation-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-http-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-io-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-security-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-server-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-servlet-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-servlets-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jetty-util-9.4.11.v20180605.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/jopt-simple-5.0.4.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka_2.12-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka_2.12-2.0.0-sources.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-clients-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-log4j-appender-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-streams-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-streams-examples-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-streams-scala_2.12-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-streams-test-utils-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/kafka-tools-2.0.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/log4j-1.2.17.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/lz4-java-1.4.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/maven-artifact-3.5.3.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/metrics-core-2.2.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/osgi-resource-locator-1.0.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/plexus-utils-3.1.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/reflections-0.9.11.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/rocksdbjni-5.7.3.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/scala-library-2.12.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/scala-logging_2.12-3.9.0.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/scala-reflect-2.12.6.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/slf4j-api-1.7.25.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/slf4j-log4j12-1.7.25.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/snappy-java-1.1.7.1.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/validation-api-1.1.0.Final.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/zkclient-0.10.jar:/usr/local/kafka_2.12-2.0.0/bin/../libs/zookeeper-3.4.13.jar (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,134] INFO Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,134] INFO Client environment:java.io.tmpdir=/tmp (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,134] INFO Client environment:java.compiler=<NA> (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,134] INFO Client environment:os.name=Linux (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,134] INFO Client environment:os.arch=amd64 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,134] INFO Client environment:os.version=3.10.0-327.el7.x86_64 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,135] INFO Client environment:user.name=root (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,135] INFO Client environment:user.home=/root (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,135] INFO Client environment:user.dir=/usr/local/kafka_2.12-2.0.0 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,137] INFO Initiating client connection, connectString=localhost:2181 sessionTimeout=6000 watcher=kafka.zookeeper.ZooKeeperClient$ZooKeeperClientWatcher$@76908cc0 (org.apache.zookeeper.ZooKeeper)
[2018-08-20 00:24:05,165] INFO [ZooKeeperClient] Waiting until connected. (kafka.zookeeper.ZooKeeperClient)
[2018-08-20 00:24:05,166] INFO Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:24:05,175] INFO Socket connection established to localhost/0:0:0:0:0:0:0:1:2181, initiating session (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:24:05,228] INFO Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x1000055ea0a0000, negotiated timeout = 6000 (org.apache.zookeeper.ClientCnxn)
[2018-08-20 00:24:05,234] INFO [ZooKeeperClient] Connected. (kafka.zookeeper.ZooKeeperClient)
[2018-08-20 00:24:06,107] INFO Cluster ID = ZSz7jNs1QSm3GfcmSMV5AQ (kafka.server.KafkaServer)
[2018-08-20 00:24:06,119] WARN No meta.properties file under dir /usr/local/kafka_2.12-2.0.0/logs/meta.properties (kafka.server.BrokerMetadataCheckpoint)
[2018-08-20 00:24:06,453] INFO KafkaConfig values: 
	advertised.host.name = null
	advertised.listeners = PLAINTEXT://localhost:9092
	advertised.port = null
	alter.config.policy.class.name = null
	alter.log.dirs.replication.quota.window.num = 11
	alter.log.dirs.replication.quota.window.size.seconds = 1
	authorizer.class.name = 
	auto.create.topics.enable = true
	auto.leader.rebalance.enable = true
	background.threads = 10
	broker.id = 0
	broker.id.generation.enable = true
	broker.rack = null
	client.quota.callback.class = null
	compression.type = producer
	connections.max.idle.ms = 600000
	controlled.shutdown.enable = true
	controlled.shutdown.max.retries = 3
	controlled.shutdown.retry.backoff.ms = 5000
	controller.socket.timeout.ms = 30000
	create.topic.policy.class.name = null
	default.replication.factor = 1
	delegation.token.expiry.check.interval.ms = 3600000
	delegation.token.expiry.time.ms = 86400000
	delegation.token.master.key = null
	delegation.token.max.lifetime.ms = 604800000
	delete.records.purgatory.purge.interval.requests = 1
	delete.topic.enable = true
	fetch.purgatory.purge.interval.requests = 1000
	group.initial.rebalance.delay.ms = 0
	group.max.session.timeout.ms = 300000
	group.min.session.timeout.ms = 6000
	host.name = 
	inter.broker.listener.name = null
	inter.broker.protocol.version = 2.0-IV1
	leader.imbalance.check.interval.seconds = 300
	leader.imbalance.per.broker.percentage = 10
	listener.security.protocol.map = PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
	listeners = PLAINTEXT://localhost:9092
	log.cleaner.backoff.ms = 15000
	log.cleaner.dedupe.buffer.size = 134217728
	log.cleaner.delete.retention.ms = 86400000
	log.cleaner.enable = true
	log.cleaner.io.buffer.load.factor = 0.9
	log.cleaner.io.buffer.size = 524288
	log.cleaner.io.max.bytes.per.second = 1.7976931348623157E308
	log.cleaner.min.cleanable.ratio = 0.5
	log.cleaner.min.compaction.lag.ms = 0
	log.cleaner.threads = 1
	log.cleanup.policy = [delete]
	log.dir = /tmp/kafka-logs
	log.dirs = /usr/local/kafka_2.12-2.0.0/logs
	log.flush.interval.messages = 9223372036854775807
	log.flush.interval.ms = null
	log.flush.offset.checkpoint.interval.ms = 60000
	log.flush.scheduler.interval.ms = 9223372036854775807
	log.flush.start.offset.checkpoint.interval.ms = 60000
	log.index.interval.bytes = 4096
	log.index.size.max.bytes = 10485760
	log.message.downconversion.enable = true
	log.message.format.version = 2.0-IV1
	log.message.timestamp.difference.max.ms = 9223372036854775807
	log.message.timestamp.type = CreateTime
	log.preallocate = false
	log.retention.bytes = -1
	log.retention.check.interval.ms = 300000
	log.retention.hours = 168
	log.retention.minutes = null
	log.retention.ms = null
	log.roll.hours = 168
	log.roll.jitter.hours = 0
	log.roll.jitter.ms = null
	log.roll.ms = null
	log.segment.bytes = 1073741824
	log.segment.delete.delay.ms = 60000
	max.connections.per.ip = 2147483647
	max.connections.per.ip.overrides = 
	max.incremental.fetch.session.cache.slots = 1000
	message.max.bytes = 1000012
	metric.reporters = []
	metrics.num.samples = 2
	metrics.recording.level = INFO
	metrics.sample.window.ms = 30000
	min.insync.replicas = 1
	num.io.threads = 8
	num.network.threads = 3
	num.partitions = 1
	num.recovery.threads.per.data.dir = 1
	num.replica.alter.log.dirs.threads = null
	num.replica.fetchers = 1
	offset.metadata.max.bytes = 4096
	offsets.commit.required.acks = -1
	offsets.commit.timeout.ms = 5000
	offsets.load.buffer.size = 5242880
	offsets.retention.check.interval.ms = 600000
	offsets.retention.minutes = 10080
	offsets.topic.compression.codec = 0
	offsets.topic.num.partitions = 50
	offsets.topic.replication.factor = 1
	offsets.topic.segment.bytes = 104857600
	password.encoder.cipher.algorithm = AES/CBC/PKCS5Padding
	password.encoder.iterations = 4096
	password.encoder.key.length = 128
	password.encoder.keyfactory.algorithm = null
	password.encoder.old.secret = null
	password.encoder.secret = null
	port = 9092
	principal.builder.class = null
	producer.purgatory.purge.interval.requests = 1000
	queued.max.request.bytes = -1
	queued.max.requests = 500
	quota.consumer.default = 9223372036854775807
	quota.producer.default = 9223372036854775807
	quota.window.num = 11
	quota.window.size.seconds = 1
	replica.fetch.backoff.ms = 1000
	replica.fetch.max.bytes = 1048576
	replica.fetch.min.bytes = 1
	replica.fetch.response.max.bytes = 10485760
	replica.fetch.wait.max.ms = 500
	replica.high.watermark.checkpoint.interval.ms = 5000
	replica.lag.time.max.ms = 10000
	replica.socket.receive.buffer.bytes = 65536
	replica.socket.timeout.ms = 30000
	replication.quota.window.num = 11
	replication.quota.window.size.seconds = 1
	request.timeout.ms = 30000
	reserved.broker.max.id = 1000
	sasl.client.callback.handler.class = null
	sasl.enabled.mechanisms = [GSSAPI]
	sasl.jaas.config = null
	sasl.kerberos.kinit.cmd = /usr/bin/kinit
	sasl.kerberos.min.time.before.relogin = 60000
	sasl.kerberos.principal.to.local.rules = [DEFAULT]
	sasl.kerberos.service.name = null
	sasl.kerberos.ticket.renew.jitter = 0.05
	sasl.kerberos.ticket.renew.window.factor = 0.8
	sasl.login.callback.handler.class = null
	sasl.login.class = null
	sasl.login.refresh.buffer.seconds = 300
	sasl.login.refresh.min.period.seconds = 60
	sasl.login.refresh.window.factor = 0.8
	sasl.login.refresh.window.jitter = 0.05
	sasl.mechanism.inter.broker.protocol = GSSAPI
	sasl.server.callback.handler.class = null
	security.inter.broker.protocol = PLAINTEXT
	socket.receive.buffer.bytes = 102400
	socket.request.max.bytes = 104857600
	socket.send.buffer.bytes = 102400
	ssl.cipher.suites = []
	ssl.client.auth = none
	ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
	ssl.endpoint.identification.algorithm = https
	ssl.key.password = null
	ssl.keymanager.algorithm = SunX509
	ssl.keystore.location = null
	ssl.keystore.password = null
	ssl.keystore.type = JKS
	ssl.protocol = TLS
	ssl.provider = null
	ssl.secure.random.implementation = null
	ssl.trustmanager.algorithm = PKIX
	ssl.truststore.location = null
	ssl.truststore.password = null
	ssl.truststore.type = JKS
	transaction.abort.timed.out.transaction.cleanup.interval.ms = 60000
	transaction.max.timeout.ms = 900000
	transaction.remove.expired.transaction.cleanup.interval.ms = 3600000
	transaction.state.log.load.buffer.size = 5242880
	transaction.state.log.min.isr = 1
	transaction.state.log.num.partitions = 50
	transaction.state.log.replication.factor = 1
	transaction.state.log.segment.bytes = 104857600
	transactional.id.expiration.ms = 604800000
	unclean.leader.election.enable = false
	zookeeper.connect = localhost:2181
	zookeeper.connection.timeout.ms = 6000
	zookeeper.max.in.flight.requests = 10
	zookeeper.session.timeout.ms = 6000
	zookeeper.set.acl = false
	zookeeper.sync.time.ms = 2000
 (kafka.server.KafkaConfig)
[2018-08-20 00:24:06,471] INFO KafkaConfig values: 
	advertised.host.name = null
	advertised.listeners = PLAINTEXT://localhost:9092
	advertised.port = null
	alter.config.policy.class.name = null
	alter.log.dirs.replication.quota.window.num = 11
	alter.log.dirs.replication.quota.window.size.seconds = 1
	authorizer.class.name = 
	auto.create.topics.enable = true
	auto.leader.rebalance.enable = true
	background.threads = 10
	broker.id = 0
	broker.id.generation.enable = true
	broker.rack = null
	client.quota.callback.class = null
	compression.type = producer
	connections.max.idle.ms = 600000
	controlled.shutdown.enable = true
	controlled.shutdown.max.retries = 3
	controlled.shutdown.retry.backoff.ms = 5000
	controller.socket.timeout.ms = 30000
	create.topic.policy.class.name = null
	default.replication.factor = 1
	delegation.token.expiry.check.interval.ms = 3600000
	delegation.token.expiry.time.ms = 86400000
	delegation.token.master.key = null
	delegation.token.max.lifetime.ms = 604800000
	delete.records.purgatory.purge.interval.requests = 1
	delete.topic.enable = true
	fetch.purgatory.purge.interval.requests = 1000
	group.initial.rebalance.delay.ms = 0
	group.max.session.timeout.ms = 300000
	group.min.session.timeout.ms = 6000
	host.name = 
	inter.broker.listener.name = null
	inter.broker.protocol.version = 2.0-IV1
	leader.imbalance.check.interval.seconds = 300
	leader.imbalance.per.broker.percentage = 10
	listener.security.protocol.map = PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
	listeners = PLAINTEXT://localhost:9092
	log.cleaner.backoff.ms = 15000
	log.cleaner.dedupe.buffer.size = 134217728
	log.cleaner.delete.retention.ms = 86400000
	log.cleaner.enable = true
	log.cleaner.io.buffer.load.factor = 0.9
	log.cleaner.io.buffer.size = 524288
	log.cleaner.io.max.bytes.per.second = 1.7976931348623157E308
	log.cleaner.min.cleanable.ratio = 0.5
	log.cleaner.min.compaction.lag.ms = 0
	log.cleaner.threads = 1
	log.cleanup.policy = [delete]
	log.dir = /tmp/kafka-logs
	log.dirs = /usr/local/kafka_2.12-2.0.0/logs
	log.flush.interval.messages = 9223372036854775807
	log.flush.interval.ms = null
	log.flush.offset.checkpoint.interval.ms = 60000
	log.flush.scheduler.interval.ms = 9223372036854775807
	log.flush.start.offset.checkpoint.interval.ms = 60000
	log.index.interval.bytes = 4096
	log.index.size.max.bytes = 10485760
	log.message.downconversion.enable = true
	log.message.format.version = 2.0-IV1
	log.message.timestamp.difference.max.ms = 9223372036854775807
	log.message.timestamp.type = CreateTime
	log.preallocate = false
	log.retention.bytes = -1
	log.retention.check.interval.ms = 300000
	log.retention.hours = 168
	log.retention.minutes = null
	log.retention.ms = null
	log.roll.hours = 168
	log.roll.jitter.hours = 0
	log.roll.jitter.ms = null
	log.roll.ms = null
	log.segment.bytes = 1073741824
	log.segment.delete.delay.ms = 60000
	max.connections.per.ip = 2147483647
	max.connections.per.ip.overrides = 
	max.incremental.fetch.session.cache.slots = 1000
	message.max.bytes = 1000012
	metric.reporters = []
	metrics.num.samples = 2
	metrics.recording.level = INFO
	metrics.sample.window.ms = 30000
	min.insync.replicas = 1
	num.io.threads = 8
	num.network.threads = 3
	num.partitions = 1
	num.recovery.threads.per.data.dir = 1
	num.replica.alter.log.dirs.threads = null
	num.replica.fetchers = 1
	offset.metadata.max.bytes = 4096
	offsets.commit.required.acks = -1
	offsets.commit.timeout.ms = 5000
	offsets.load.buffer.size = 5242880
	offsets.retention.check.interval.ms = 600000
	offsets.retention.minutes = 10080
	offsets.topic.compression.codec = 0
	offsets.topic.num.partitions = 50
	offsets.topic.replication.factor = 1
	offsets.topic.segment.bytes = 104857600
	password.encoder.cipher.algorithm = AES/CBC/PKCS5Padding
	password.encoder.iterations = 4096
	password.encoder.key.length = 128
	password.encoder.keyfactory.algorithm = null
	password.encoder.old.secret = null
	password.encoder.secret = null
	port = 9092
	principal.builder.class = null
	producer.purgatory.purge.interval.requests = 1000
	queued.max.request.bytes = -1
	queued.max.requests = 500
	quota.consumer.default = 9223372036854775807
	quota.producer.default = 9223372036854775807
	quota.window.num = 11
	quota.window.size.seconds = 1
	replica.fetch.backoff.ms = 1000
	replica.fetch.max.bytes = 1048576
	replica.fetch.min.bytes = 1
	replica.fetch.response.max.bytes = 10485760
	replica.fetch.wait.max.ms = 500
	replica.high.watermark.checkpoint.interval.ms = 5000
	replica.lag.time.max.ms = 10000
	replica.socket.receive.buffer.bytes = 65536
	replica.socket.timeout.ms = 30000
	replication.quota.window.num = 11
	replication.quota.window.size.seconds = 1
	request.timeout.ms = 30000
	reserved.broker.max.id = 1000
	sasl.client.callback.handler.class = null
	sasl.enabled.mechanisms = [GSSAPI]
	sasl.jaas.config = null
	sasl.kerberos.kinit.cmd = /usr/bin/kinit
	sasl.kerberos.min.time.before.relogin = 60000
	sasl.kerberos.principal.to.local.rules = [DEFAULT]
	sasl.kerberos.service.name = null
	sasl.kerberos.ticket.renew.jitter = 0.05
	sasl.kerberos.ticket.renew.window.factor = 0.8
	sasl.login.callback.handler.class = null
	sasl.login.class = null
	sasl.login.refresh.buffer.seconds = 300
	sasl.login.refresh.min.period.seconds = 60
	sasl.login.refresh.window.factor = 0.8
	sasl.login.refresh.window.jitter = 0.05
	sasl.mechanism.inter.broker.protocol = GSSAPI
	sasl.server.callback.handler.class = null
	security.inter.broker.protocol = PLAINTEXT
	socket.receive.buffer.bytes = 102400
	socket.request.max.bytes = 104857600
	socket.send.buffer.bytes = 102400
	ssl.cipher.suites = []
	ssl.client.auth = none
	ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
	ssl.endpoint.identification.algorithm = https
	ssl.key.password = null
	ssl.keymanager.algorithm = SunX509
	ssl.keystore.location = null
	ssl.keystore.password = null
	ssl.keystore.type = JKS
	ssl.protocol = TLS
	ssl.provider = null
	ssl.secure.random.implementation = null
	ssl.trustmanager.algorithm = PKIX
	ssl.truststore.location = null
	ssl.truststore.password = null
	ssl.truststore.type = JKS
	transaction.abort.timed.out.transaction.cleanup.interval.ms = 60000
	transaction.max.timeout.ms = 900000
	transaction.remove.expired.transaction.cleanup.interval.ms = 3600000
	transaction.state.log.load.buffer.size = 5242880
	transaction.state.log.min.isr = 1
	transaction.state.log.num.partitions = 50
	transaction.state.log.replication.factor = 1
	transaction.state.log.segment.bytes = 104857600
	transactional.id.expiration.ms = 604800000
	unclean.leader.election.enable = false
	zookeeper.connect = localhost:2181
	zookeeper.connection.timeout.ms = 6000
	zookeeper.max.in.flight.requests = 10
	zookeeper.session.timeout.ms = 6000
	zookeeper.set.acl = false
	zookeeper.sync.time.ms = 2000
 (kafka.server.KafkaConfig)
[2018-08-20 00:24:06,581] INFO [ThrottledChannelReaper-Fetch]: Starting (kafka.server.ClientQuotaManager$ThrottledChannelReaper)
[2018-08-20 00:24:06,581] INFO [ThrottledChannelReaper-Produce]: Starting (kafka.server.ClientQuotaManager$ThrottledChannelReaper)
[2018-08-20 00:24:06,583] INFO [ThrottledChannelReaper-Request]: Starting (kafka.server.ClientQuotaManager$ThrottledChannelReaper)
[2018-08-20 00:24:06,738] INFO Loading logs. (kafka.log.LogManager)
[2018-08-20 00:24:06,752] INFO Logs loading complete in 14 ms. (kafka.log.LogManager)
[2018-08-20 00:24:06,777] INFO Starting log cleanup with a period of 300000 ms. (kafka.log.LogManager)
[2018-08-20 00:24:06,785] INFO Starting log flusher with a default period of 9223372036854775807 ms. (kafka.log.LogManager)
[2018-08-20 00:24:08,659] INFO Awaiting socket connections on localhost:9092. (kafka.network.Acceptor)
[2018-08-20 00:24:08,724] INFO [SocketServer brokerId=0] Started 1 acceptor threads (kafka.network.SocketServer)
[2018-08-20 00:24:08,783] INFO [ExpirationReaper-0-Produce]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-08-20 00:24:08,788] INFO [ExpirationReaper-0-DeleteRecords]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-08-20 00:24:08,783] INFO [ExpirationReaper-0-Fetch]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-08-20 00:24:08,842] INFO [LogDirFailureHandler]: Starting (kafka.server.ReplicaManager$LogDirFailureHandler)
[2018-08-20 00:24:08,894] INFO Creating /brokers/ids/0 (is it secure? false) (kafka.zk.KafkaZkClient)
[2018-08-20 00:24:08,898] INFO Result of znode creation at /brokers/ids/0 is: OK (kafka.zk.KafkaZkClient)
[2018-08-20 00:24:08,901] INFO Registered broker 0 at path /brokers/ids/0 with addresses: ArrayBuffer(EndPoint(localhost,9092,ListenerName(PLAINTEXT),PLAINTEXT)) (kafka.zk.KafkaZkClient)
[2018-08-20 00:24:08,905] WARN No meta.properties file under dir /usr/local/kafka_2.12-2.0.0/logs/meta.properties (kafka.server.BrokerMetadataCheckpoint)
[2018-08-20 00:24:09,002] INFO [ExpirationReaper-0-topic]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-08-20 00:24:09,005] INFO Creating /controller (is it secure? false) (kafka.zk.KafkaZkClient)
[2018-08-20 00:24:09,008] INFO [ExpirationReaper-0-Heartbeat]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-08-20 00:24:09,057] INFO [ExpirationReaper-0-Rebalance]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-08-20 00:24:09,102] INFO Result of znode creation at /controller is: OK (kafka.zk.KafkaZkClient)
[2018-08-20 00:24:09,122] INFO [GroupCoordinator 0]: Starting up. (kafka.coordinator.group.GroupCoordinator)
[2018-08-20 00:24:09,128] INFO [GroupCoordinator 0]: Startup complete. (kafka.coordinator.group.GroupCoordinator)
[2018-08-20 00:24:09,161] INFO [GroupMetadataManager brokerId=0] Removed 0 expired offsets in 34 milliseconds. (kafka.coordinator.group.GroupMetadataManager)
[2018-08-20 00:24:09,228] INFO [ProducerId Manager 0]: Acquired new producerId block (brokerId:0,blockStartProducerId:0,blockEndProducerId:999) by writing to Zk with path version 1 (kafka.coordinator.transaction.ProducerIdManager)
[2018-08-20 00:24:09,555] INFO [TransactionCoordinator id=0] Starting up. (kafka.coordinator.transaction.TransactionCoordinator)
[2018-08-20 00:24:09,558] INFO [TransactionCoordinator id=0] Startup complete. (kafka.coordinator.transaction.TransactionCoordinator)
[2018-08-20 00:24:09,576] INFO [Transaction Marker Channel Manager 0]: Starting (kafka.coordinator.transaction.TransactionMarkerChannelManager)
[2018-08-20 00:24:09,778] INFO [/config/changes-event-process-thread]: Starting (kafka.common.ZkNodeChangeNotificationListener$ChangeEventProcessThread)
[2018-08-20 00:24:09,870] INFO [SocketServer brokerId=0] Started processors for 1 acceptors (kafka.network.SocketServer)
[2018-08-20 00:24:09,930] INFO Kafka version : 2.0.0 (org.apache.kafka.common.utils.AppInfoParser)
[2018-08-20 00:24:09,930] INFO Kafka commitId : 3402a8361b734732 (org.apache.kafka.common.utils.AppInfoParser)
[2018-08-20 00:24:09,959] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```

### 停止Kafka

停止`Kafka`命令：

```
[root@localhost kafka_2.12-2.0.0]# ./bin/kafka-server-stop.sh
```

### 创建Topic

创建一个名为`test`的Topic（只有一个副本，一个分区)，创建并验证Topic:

```
[root@localhost bin]# pwd
/usr/local/kafka_2.12-2.0.0/bin
[root@localhost bin]# ./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
Created topic "test".
```

### 查看Topic

列出所有的Topic：

```
[root@localhost bin]# ./kafka-topics.sh --list --zookeeper localhost:2181
test
```

```
[root@localhost bin]# ./kafka-topics.sh --zookeeper localhost:2181 --describe --topic test
Topic:test	PartitionCount:1	ReplicationFactor:1	Configs:
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```

### 发布消息

向Topic上发布消息，按`Ctrl+D`结束：

```
[root@localhost bin]# ./kafka-console-producer.sh --broker-list localhost:9092 --topic test
>My name is song.
>Hello Kafka.
```

### 接收消息

从Topic上接收消息，按`Ctrl+C`结束：

```
[root@localhost bin]# ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
My name is song.
Hello Kafka.
^CProcessed a total of 2 messages
```

## broker配置



### 常规配置



* [参考：在CentOS7上安装Zookeeper-3.4.9服务](https://www.linuxidc.com/Linux/2016-09/135052.htm)
* [参考：Linux系统中Kafka环境的搭建](https://blog.csdn.net/xuzhelin/article/details/71515208)
* [参考：Kafka配置文件server.properties](https://www.cnblogs.com/LUA123/p/7349145.html)
* [参考：kafka学习(1)linux下的安装和启动，以及Java示例代码](https://blog.csdn.net/kkgbn/article/details/77540491)
* [参考：搭建 Zookeeper-3.4.11 集群](https://www.cnblogs.com/LUA123/p/7222216.html)