---
layout: post
title: "kafka-manager工具"
categories: 大数据
tags: 大数据 apache Zookeeper Kafka MQ Linux
author: Kopite
---

* content
{:toc}


本文介绍`kafka-manager`工具的使用。



## 安装kafka-manager

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

使用`/bin/kafka-manager`启动`kafka-manager`。

* [参考：kafka-manager-1.3.3.18（已编译，上传修改zookeeper直接使用）](https://blog.csdn.net/qq_20793353/article/details/81115216)
* [参考：kafka集群管理工具kafka-manager部署安装](https://www.cnblogs.com/dadonggg/p/8205302.html)
