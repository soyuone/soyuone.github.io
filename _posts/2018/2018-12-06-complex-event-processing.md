---
layout: post
title: "复杂事件处理（Complex Event Processing）"
categories: 大数据
tags: 大数据 CEP Esper
author: Kopite
---

* content
{:toc}


本文介绍`kafka-manager`工具的使用。



## SOA

`面向服务架构（Service-Oriented Architecture，SOA）`

## EDA

`事件驱动架构（Event-Driven Architecture，EDA）`

从[kafka-manager GitHub](https://github.com/yahoo/kafka-manager)下载的源码包需要编译，此处使用已经编译好的`kafka-manager-1.3.3.18.zip`，安装步骤如下：
* 使用[Bitvise SSH Client](https://www.bitvise.com/ssh-client)工具将编译好的`kafka-manager-1.3.3.18.zip`复制到`/usr/local/`路径，此处使用`192.168.80.129`
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

* [参考：复杂事件处理（Complex Event Processing）](https://blog.csdn.net/u010945683/article/details/79826372)
* [参考：复合事件处理（Complex Event Processing）介绍](https://www.cnblogs.com/shanyou/archive/2010/09/16/cep.html)
