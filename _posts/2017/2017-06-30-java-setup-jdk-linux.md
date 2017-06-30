---
layout: post
title: "jdk在Linux平台的安装和配置"
categories: java
tags: java Linux CentOS
author: Kopite
---

* content
{:toc}


jdk`版本号1.7.0_80`在Linux平台的安装和配置。



## 安装

* 到oracle官网下载[jdk-7u80-linux-x64.rpm](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html)
* 使用[Bitvise SSH Client](https://www.bitvise.com/ssh-client)工具将下载的rpm包复制到`/usr/local/`路径，复制完毕后到查看该路径下是否有rpm包

```
[root@localhost local]# ls -l
总用量 134856
drwxr-xr-x. 2 root root         6 8月  12 2015 bin
drwxr-xr-x. 2 root root         6 8月  12 2015 etc
drwxr-xr-x. 2 root root         6 8月  12 2015 games
drwxr-xr-x. 2 root root         6 8月  12 2015 include
-rw-r--r--. 1 root root 138090286 6月  30 18:03 jdk-7u80-linux-x64.rpm
drwxr-xr-x. 2 root root         6 8月  12 2015 lib
drwxr-xr-x. 2 root root         6 8月  12 2015 lib64
drwxr-xr-x. 2 root root         6 8月  12 2015 libexec
drwxr-xr-x. 2 root root         6 8月  12 2015 sbin
drwxr-xr-x. 5 root root        46 6月  20 19:37 share
drwxr-xr-x. 2 root root         6 8月  12 2015 src
```

* 查看rpm包打算把包中文件安装的位置：

```
[root@localhost local]# rpm -qlp jdk-7u80-linux-x64.rpm 
/etc
/etc/.java
/etc/.java/.systemPrefs
/etc/.java/.systemPrefs/.system.lock
/etc/.java/.systemPrefs/.systemRootModFile
/etc/init.d/jexec
/usr
/usr/java
/usr/java/jdk1.7.0_80
/usr/java/jdk1.7.0_80/COPYRIGHT
/usr/java/jdk1.7.0_80/LICENSE
/usr/java/jdk1.7.0_80/README.html
/usr/java/jdk1.7.0_80/THIRDPARTYLICENSEREADME-JAVAFX.txt
/usr/java/jdk1.7.0_80/THIRDPARTYLICENSEREADME.txt
/usr/java/jdk1.7.0_80/bin
/usr/java/jdk1.7.0_80/bin/ControlPanel
/usr/java/jdk1.7.0_80/bin/appletviewer
/usr/java/jdk1.7.0_80/bin/apt
/usr/java/jdk1.7.0_80/bin/extcheck
...
```

* 安装jdk：

```
[root@localhost local]# rpm -ivh jdk-7u80-linux-x64.rpm
准备中...                          ################################# [100%]
正在升级/安装...
   1:jdk-2000:1.7.0_80-fcs            ################################# [100%]
Unpacking JAR files...
	rt.jar...
	jsse.jar...
	charsets.jar...
	tools.jar...
	localedata.jar...
	jfxrt.jar...
```

* 查看rpm包中文件的安装位置：

```
[root@localhost ~]# rpm -ql jdk
/etc
/etc/.java
/etc/.java/.systemPrefs
/etc/.java/.systemPrefs/.system.lock
/etc/.java/.systemPrefs/.systemRootModFile
/etc/init.d/jexec
/usr
/usr/java
/usr/java/jdk1.7.0_80
/usr/java/jdk1.7.0_80/COPYRIGHT
/usr/java/jdk1.7.0_80/LICENSE
/usr/java/jdk1.7.0_80/README.html
/usr/java/jdk1.7.0_80/THIRDPARTYLICENSEREADME-JAVAFX.txt
/usr/java/jdk1.7.0_80/THIRDPARTYLICENSEREADME.txt
/usr/java/jdk1.7.0_80/bin
/usr/java/jdk1.7.0_80/bin/ControlPanel
/usr/java/jdk1.7.0_80/bin/appletviewer
/usr/java/jdk1.7.0_80/bin/apt
/usr/java/jdk1.7.0_80/bin/extcheck
...
```

## 配置环境变量

* 打开系统环境变量文件：

```
[root@localhost etc]# vi /etc/profile
```

* 以追加方式添加如下内容：

```
JAVA_HOME=/usr/java/jdk1.7.0_80
JRE_HOME=/usr/java/jdk1.7.0_80/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

* 使修改立即生效：

```
[root@localhost etc]# source /etc/profile
```

* 查看PATH环境变量的值：

```
[root@localhost etc]# echo $PATH
/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/root/bin:/usr/java/jdk1.7.0_80/bin:/usr/java/jdk1.7.0_80/jre/bin
```

* 验证：

```
[root@localhost ~]# java
```

```
[root@localhost ~]# javac
```

```
[root@localhost ~]# java -version
java version "1.7.0_80"
Java(TM) SE Runtime Environment (build 1.7.0_80-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)
```

* 删除`/usr/local/`路径下的rpm安装包：

```
[root@localhost local]# rm -rf jdk-7u80-linux-x64.rpm 
```

## 卸载

查询Linux中安装的jdk：

```
[root@localhost ~]# rpm -qa | grep jdk
java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64
java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64
java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64
```

使用`yum`命令卸载上述安装的jdk：

```
[root@localhost ~]# yum -y remove java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64
已加载插件：fastestmirror, langpacks
正在解决依赖关系
--> 正在检查事务
---> 软件包 java-1.7.0-openjdk-headless.x86_64.1.1.7.0.91-2.6.2.3.el7 将被 删除
--> 正在处理依赖关系 java-1.7.0-openjdk-headless = 1:1.7.0.91-2.6.2.3.el7，它被软件包 1:java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64 需要
--> 正在检查事务
---> 软件包 java-1.7.0-openjdk.x86_64.1.1.7.0.91-2.6.2.3.el7 将被 删除
--> 解决依赖关系完成
base/7/x86_64                                            | 3.6 kB     00:00     
extras/7/x86_64                                          | 3.4 kB     00:00     
updates/7/x86_64                                         | 3.4 kB     00:00     
updates/7/x86_64/primary_db                              | 7.1 MB     00:02     

依赖关系解决

================================================================================
 Package                      架构    版本                     源          大小
================================================================================
正在删除:
 java-1.7.0-openjdk-headless  x86_64  1:1.7.0.91-2.6.2.3.el7   @anaconda   91 M
为依赖而移除:
 java-1.7.0-openjdk           x86_64  1:1.7.0.91-2.6.2.3.el7   @anaconda  493 k

事务概要
================================================================================
移除  1 软件包 (+1 依赖软件包)

安装大小：91 M
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在删除    : 1:java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64            1/2 
  正在删除    : 1:java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64   2/2 
  验证中      : 1:java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64   1/2 
  验证中      : 1:java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64            2/2 

删除:
  java-1.7.0-openjdk-headless.x86_64 1:1.7.0.91-2.6.2.3.el7                     

作为依赖被删除:
  java-1.7.0-openjdk.x86_64 1:1.7.0.91-2.6.2.3.el7                              

完毕！
```

* [参考：在CentOS中安装与配置jdk](http://blog.csdn.net/czmchen/article/details/41047187)
* [参考：在CentOS中安装jdk的三种方法](http://www.linuxidc.com/Linux/2016-09/134941.htm)