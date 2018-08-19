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

![](/image/2018/2018-08-19-kafka-setup-linux-1.png)

`Kafka`使用`Zookeeper`保存集群的元数据信息和消费者信息，见上图所示，`Zookeeper`的安装步骤如下（`Kafka`发行版自带了`Zookeeper`，可直接从脚本启动）：
* 进入[Apache Zookeeper官方网站](http://zookeeper.apache.org/)，选择合适的镜像站点及Apache Zookeeper版本进行下载，此处使用[http://www-eu.apache.org/dist/zookeeper/](http://www-eu.apache.org/dist/zookeeper/)镜像站点进行下载，选择当前的稳定版`3.4.12版本`，找到`zookeeper-3.4.12.tar.gz`，点击即可下载
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
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.12/
export PATH=$ZOOKEEPER_HOME/bin:$PATH
export PATH
# zookeeper-3.4.12 config end
```

* 使`/etc/profile`文件立即生效：

```
[root@ etc]# source /etc/profile
```

* 输入`zkServer.sh start`，启动`Zookeeper`服务，如打印如下信息则表示启动成功：

```
[root@ zookeeper-3.4.12]# zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

* 查询`Zookeeper`状态：

```
[root@ zookeeper-3.4.12]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: standalone
```

* 停止`Zookeeper`服务：

```
[root@ zookeeper-3.4.12]# zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.12/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```

* 重启`Zookeeper`服务：

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

## Zookeeper群组

`Zookeeper`集群被称为群组。`Zookeeper`使用的是一致性协议，所以建议每个节点里应该包含奇数个节点，因为只有当群组里的大多数节点处于可用状态，`Zookeeper`才能处理外部的请求。

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
# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
# 套接字服务器监听的地址。如果没有配置，就使用
# java.net.InetAddress.getCanonicalHostName()的返回值
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://localhost:9092

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
# 节点的主机名会通知给生产者和消费者。如果没有配置，"listeners"配置了就使
# 用"listeners"的值，否则就使用java.net.InetAddress.getCanonicalHostName()的返回值
advertised.listeners=PLAINTEXT://localhost:9092
```

```
# A comma separated list of directories under which to store log files
# 日志目录
log.dirs=/usr/local/kafka_2.12-2.0.0/logs
```

* 输入`sh ./bin/kafka-server-start.sh ./config/server.properties`启动`Kafka`，提示信息报错，原因`Java`版本过低，升级`Java`版本为1.8

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

* [参考：在CentOS7上安装Zookeeper-3.4.9服务](https://www.linuxidc.com/Linux/2016-09/135052.htm)
* [参考：Linux系统中Kafka环境的搭建](https://blog.csdn.net/xuzhelin/article/details/71515208)
* [参考：Kafka配置文件server.properties](https://www.cnblogs.com/LUA123/p/7349145.html)
* [参考：kafka学习(1)linux下的安装和启动，以及Java示例代码](https://blog.csdn.net/kkgbn/article/details/77540491)