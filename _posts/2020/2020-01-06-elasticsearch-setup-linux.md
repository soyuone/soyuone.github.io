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

## 启动Elasticsearch

从[Elasticsearch](https://www.elastic.co/cn/)下载的源码包需要编译，此处使用已经编译好的`Elasticsearch`的安装步骤如下：
* 进入使用[Bitvise SSH Client](https://www.bitvise.com/ssh-client)工具将编译好的`kafka-manager-1.3.3.18.zip`复制到`/usr/local/`路径，此处使用`192.168.80.129`
* 将`kafka-manager-1.3.3.18.zip`包解压至`/usr/local/`路径：

```
[root@localhost local]# unzip kafka-manager-1.3.3.18.zip
Archive:  kafka-manager-1.3.3.18.zip
  inflating: kafka-manager-1.3.3.18/lib/kafka-manager.kafka-manager-1.3.3.18-sans-externalized.jar  
  inflating: kafka-manager-1.3.3.18/lib/com.typesafe.play.twirl-api_2.11-1.1.1.jar  
  inflating: kafka-manager-1.3.3.18/lib/org.apache.commons.commons-lang3-3.4.jar  
  inflating: kafka-manager-1.3.3.18/lib/com.typesafe.play.play-server_2.11-2.4.6.jar  
  inflating: kafka-manager-1.3.3.18/lib/com.typesafe.play.play_2.11-2.4.6.jar  
  inflating: kafka-manager-1.3.3.18/lib/com.typesafe.play.build-link-2.4.6.jar  
  inflating: kafka-manager-1.3.3.18/lib/com.typesafe.play.play-exceptions-2.4.6.jar  
  inflating: kafka-manager-1.3.3.18/lib/org.javassist.javassist-3.19.0-GA.jar  
  inflating: kafka-manager-1.3.3.18/lib/com.typesafe.play.play-iteratees_2.11-2.4.6.jar  
  inflating: kafka-manager-1.3.3.18/lib/org.scala-stm.scala-stm_2.11-0.7.jar
...
```

* 进入`/usr/local/kafka-manager-1.3.3.18/conf`目录，修改`application.conf`，保存并关闭：

```
kafka-manager.zkhosts="192.168.80.129:2181,192.168.80.130:2181,192.168.80.131:2181"
```

## 启动kafka-manager

`kafka-manager`默认访问端口是`9000`，开放此端口：

```
[root@localhost conf]# firewall-cmd --query-port=9000/tcp
no
[root@localhost conf]# firewall-cmd --permanent --zone=public --add-port=9000/tcp
success
[root@localhost conf]# firewall-cmd --reload
success
[root@localhost conf]# firewall-cmd --query-port=9000/tcp
yes
```

使用以下命令启动`kafka-manager`，浏览器中输入`http://192.168.80.129:9000`即可访问`kafka-manager`：

```
[root@localhost bin]# ./kafka-manager
```

或者

```
[root@localhost bin]# nohup ./kafka-manager &
```

当`kafka-manager`启动异常时，删掉`application.home_IS_UNDEFINED`、`RUNNING_PID`、`nohup.out`后再次执行上述启动命令：

```
[root@localhost kafka-manager-1.3.3.18]# rm -rf application.home_IS_UNDEFINED/
[root@localhost kafka-manager-1.3.3.18]# rm -rf RUNNING_PID
```

## 关闭kafka-manager

使用`ps -ef|grep kafka-manager`命令或者`jps`命令查看`kafka-manager`进程，然后杀掉进程即可：

```
[root@localhost bin]# ps -ef|grep kafka-manager
root      9176  5907  8 15:03 pts/1    00:01:07 /usr/java/jdk1.8.0_181-amd64/bin/java -Duser.dir=/usr/local/kafka-manager-1.3.3.18 -cp /usr/local/kafka-manager-1.3.3.18/lib/../conf/:/usr/local/kafka-manager-1.3.3.18/lib/kafka-manager.kafka-manager-1.3.3.18-sans-externalized.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.twirl-api_2.11-1.1.1.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.commons.commons-lang3-3.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.play-server_2.11-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.play_2.11-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.build-link-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.play-exceptions-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.javassist.javassist-3.19.0-GA.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.play-iteratees_2.11-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.scala-stm.scala-stm_2.11-0.7.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.config-1.3.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.play-json_2.11-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.play-functional_2.11-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.play-datacommons_2.11-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/joda-time.joda-time-2.8.1.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.joda.joda-convert-1.7.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.fasterxml.jackson.datatype.jackson-datatype-jdk8-2.5.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.fasterxml.jackson.datatype.jackson-datatype-jsr310-2.5.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.play-netty-utils-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.slf4j.jul-to-slf4j-1.7.12.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.slf4j.jcl-over-slf4j-1.7.12.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.scala-lang.modules.scala-parser-combinators_2.11-1.0.1.jar:/usr/local/kafka-manager-1.3.3.18/lib/ch.qos.logback.logback-core-1.1.3.jar:/usr/local/kafka-manager-1.3.3.18/lib/ch.qos.logback.logback-classic-1.1.3.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.akka.akka-actor_2.11-2.3.14.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.akka.akka-slf4j_2.11-2.3.14.jar:/usr/localkafka-manager-1.3.3.18/lib/commons-codec.commons-codec-1.10.jar:/usr/local/kafka-manager-1.3.3.18/lib/xerces.xercesImpl-2.11.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/xml-apis.xml-apis-1.4.01.jar:/usr/local/kafka-manager-1.3.3.18/lib/javax.transaction.jta-1.1.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.google.inject.guice-4.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/javax.inject.javax.inject-1.jar:/usr/local/kafka-manager-1.3.3.18/lib/aopalliance.aopalliance-1.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.google.guava.guava-16.0.1.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.google.inject.extensions.guice-assistedinject-4.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.play.play-netty-server_2.11-2.4.6.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.netty.netty-http-pipelining-1.1.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.google.code.findbugs.jsr305-2.0.1.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.webjars-play_2.11-2.4.0-2.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.requirejs-2.1.20.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.webjars-locator-0.28.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.webjars-locator-core-0.27.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.commons.commons-compress-1.9.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.npm.validate.js-0.8.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.bootstrap-3.3.5.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.jquery-2.1.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.backbonejs-1.2.3.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.underscorejs-1.8.3.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.dustjs-linkedin-2.6.1-1.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.webjars.json-20121008-1.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.curator.curator-framework-2.10.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.curator.curator-client-2.10.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/jline.jline-0.9.94.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.curator.curator-recipes-2.10.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.json4s.json4s-jackson_2.11-3.4.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.json4s.json4s-core_2.11-3.4.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.json4s.json4s-ast_2.11-3.4.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.json4s.json4s-scalap_2.11-3.4.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.thoughtworks.paranamer.paranamer-2.8.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.scala-lang.modules.scala-xml_2.11-1.0.5.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.json4s.json4s-scalaz_2.11-3.4.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.scalaz.scalaz-core_2.11-7.2.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.slf4j.log4j-over-slf4j-1.7.12.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.adrianhurt.play-bootstrap3_2.11-0.4.5-P24.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.clapper.grizzled-slf4j_2.11-1.0.2.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.kafka.kafka_2.11-1.1.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.kafka.kafka-clients-1.1.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.lz4.lz4-java-1.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.xerial.snappy.snappy-java-1.1.7.1.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.slf4j.slf4j-api-1.7.25.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.fasterxml.jackson.core.jackson-databind-2.9.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.fasterxml.jackson.core.jackson-annotations-2.9.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.fasterxml.jackson.core.jackson-core-2.9.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/net.sf.jopt-simple.jopt-simple-5.0.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.yammer.metrics.metrics-core-2.2.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.scala-lang.scala-library-2.11.12.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.scala-lang.scala-reflect-2.11.12.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.typesafe.scala-logging.scala-logging_2.11-3.7.2.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.101tec.zkclient-0.10.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.zookeeper.zookeeper-3.4.10.jar:/usr/local/kafka-manager-1.3.3.18/lib/io.netty.netty-3.10.5.Final.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.kafka.kafka-streams-1.1.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.kafka.connect-json-1.1.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.apache.kafka.connect-api-1.1.0.jar:/usr/local/kafka-manager-1.3.3.18/lib/org.rocksdb.rocksdbjni-5.7.3.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.beachape.enumeratum_2.11-1.4.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.beachape.enumeratum-macros_2.11-1.4.4.jar:/usr/local/kafka-manager-1.3.3.18/lib/com.github.ben-manes.caffeine.caffeine-2.6.2.jar:/usr/local/kafka-manager-1.3.3.18/lib/kafka-manager.kafka-manager-1.3.3.18-assets.jar play.core.server.ProdServerStart
root     10007  5907  2 15:15 pts/1    00:00:00 grep --color=auto kafka-manager
[root@localhost bin]# jps
9176 ProdServerStart
4427 QuorumPeerMain
10011 Jps
5247 Kafka
```

## 使用kafka-manager

### Add Cluster

![](/image/2018/2018-12-03-kafka-manager-1.PNG)

点击`Cluster`->`Add Cluster`，进入上图中所示的新增集群界面，依次输入`Cluster Name`、`Cluster Zookeeper Hosts`、`Kafka Version`，点击`Save`保存，新增集群后的界面如下图所示：

![](/image/2018/2018-12-03-kafka-manager-2.PNG)

点击集群名称进入`Summary`界面：

![](/image/2018/2018-12-03-kafka-manager-3.PNG)

在`Summary`界面可查看`Topics`、`Brokers`：

![](/image/2018/2018-12-03-kafka-manager-4.PNG)

![](/image/2018/2018-12-03-kafka-manager-5.PNG)

### Create Topic

![](/image/2018/2018-12-03-kafka-manager-6.PNG)

点击`Topic`->`Create Topic`，进入上图中所示的新增主题界面，依次输入`Topic`、`Partitions`、`Replication Factor`，点击`Create`保存，新增主题后的界面如下图所示：

![](/image/2018/2018-12-03-kafka-manager-7.PNG)

* [参考：kafka-manager维护工具使用指南](https://blog.csdn.net/xhpscdx/article/details/76670209)
* [参考：kafka集群管理工具kafka-manager部署安装](https://www.cnblogs.com/dadonggg/p/8205302.html)
