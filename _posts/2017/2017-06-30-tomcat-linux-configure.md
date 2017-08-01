---
layout: post
title: "tomcat在Linux平台的安装和配置"
categories: 服务器
tags: 服务器 tomcat Linux CentOS
author: Kopite
---

* content
{:toc}


tomcat`版本号Apache Tomcat/7.0.78`在Linux平台的安装和配置，安装tomcat之前需要预先安装jdk。



## 安装

### 准备工作

* 到tomcat官网下载[apache-tomcat-7.0.78.tar.gz](http://tomcat.apache.org/)
* 使用[Bitvise SSH Client](https://www.bitvise.com/ssh-client)工具将下载的`apache-tomcat-7.0.78.tar.gz`包复制到`/usr/local/`路径，复制完毕后查看该路径下是否有文件：

```
[root@localhost local]# ls -l
总用量 8760
-rw-r--r--. 1 root root 8968516 6月  30 19:41 apache-tomcat-7.0.78.tar.gz
drwxr-xr-x. 2 root root       6 8月  12 2015 bin
drwxr-xr-x. 2 root root       6 8月  12 2015 etc
drwxr-xr-x. 2 root root       6 8月  12 2015 games
drwxr-xr-x. 2 root root       6 8月  12 2015 include
drwxr-xr-x. 2 root root       6 8月  12 2015 lib
drwxr-xr-x. 2 root root       6 8月  12 2015 lib64
drwxr-xr-x. 2 root root       6 8月  12 2015 libexec
drwxr-xr-x. 2 root root       6 8月  12 2015 sbin
drwxr-xr-x. 5 root root      46 6月  20 19:37 share
drwxr-xr-x. 2 root root       6 8月  12 2015 src
```

### 解压

* 将`apache-tomcat-7.0.78.tar.gz`包解压至`/usr/local/`路径：

```
[root@localhost local]# tar -zxvf apache-tomcat-7.0.78.tar.gz
apache-tomcat-7.0.78/bin/catalina.sh
apache-tomcat-7.0.78/bin/configtest.sh
apache-tomcat-7.0.78/bin/daemon.sh
apache-tomcat-7.0.78/bin/digest.sh
apache-tomcat-7.0.78/bin/setclasspath.sh
apache-tomcat-7.0.78/bin/shutdown.sh
apache-tomcat-7.0.78/bin/startup.sh
apache-tomcat-7.0.78/bin/tool-wrapper.sh
apache-tomcat-7.0.78/bin/version.sh
apache-tomcat-7.0.78/conf/
apache-tomcat-7.0.78/conf/catalina.policy
apache-tomcat-7.0.78/conf/catalina.properties
apache-tomcat-7.0.78/conf/context.xml
apache-tomcat-7.0.78/conf/logging.properties
apache-tomcat-7.0.78/conf/server.xml
apache-tomcat-7.0.78/conf/tomcat-users.xml
apache-tomcat-7.0.78/conf/web.xml
apache-tomcat-7.0.78/bin/
apache-tomcat-7.0.78/lib/
apache-tomcat-7.0.78/logs/
...
```

### 启动服务

* 使用`apache-tomcat-7.0.78/bin/`路径下的`startup.sh`命令启动tomcat：

```
[root@localhost /]# /usr/local/apache-tomcat-7.0.78/bin/startup.sh 
Using CATALINA_BASE:   /usr/local/apache-tomcat-7.0.78
Using CATALINA_HOME:   /usr/local/apache-tomcat-7.0.78
Using CATALINA_TMPDIR: /usr/local/apache-tomcat-7.0.78/temp
Using JRE_HOME:        /usr/java/jdk1.7.0_80/jre
Using CLASSPATH:       /usr/local/apache-tomcat-7.0.78/bin/bootstrap.jar:/usr/local/apache-tomcat-7.0.78/bin/tomcat-juli.jar
Tomcat started.
```

* 浏览器中输入`http://localhost:8080/`，查看是否显示tomcat默认首页

### 停止服务

* 使用`apache-tomcat-7.0.78/bin/`路径下的`shutdown.sh`命令停止tomcat：

```
[root@localhost /]# /usr/local/apache-tomcat-7.0.78/bin/shutdown.sh 
Using CATALINA_BASE:   /usr/local/apache-tomcat-7.0.78
Using CATALINA_HOME:   /usr/local/apache-tomcat-7.0.78
Using CATALINA_TMPDIR: /usr/local/apache-tomcat-7.0.78/temp
Using JRE_HOME:        /usr/java/jdk1.7.0_80/jre
Using CLASSPATH:       /usr/local/apache-tomcat-7.0.78/bin/bootstrap.jar:/usr/local/apache-tomcat-7.0.78/bin/tomcat-juli.jar
```

### 解决中文参数乱码

* 为防止`GET`请求时的中文参数乱码，在tomcat停止状态下修改`/usr/local/apache-tomcat-7.0.78/conf`路径下的`server.xml`文件，在如下位置添加`URIEncoding="UTF-8"`：

```xml
<Connector URIEncoding="UTF-8"  port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

### 8080端口

* 此时，除本机之外的机器无法访问tomcat，查看防火墙是否开放`8080`端口，`firewall-cmd --query-port=8080/tcp`：

```
[root@localhost ~]# firewall-cmd --query-port=8080/tcp
no
```

`8080`端口并没有开放，增加`8080`端口到防火墙配置文件`/etc/sysconfig/iptables`，查找后，在上述路径并没有`iptables`文件：

```
[root@localhost sysconfig]# ls
atd              ip6tables-config  nfs            samba
authconfig       iptables-config   ntpd           saslauthd
autofs           irqbalance        ntpdate        selinux
cbq              kdump             pluto          smartmontools
cgred            kernel            qemu-ga        sshd
console          ksm               radvd          svnserve
cpupower         libvirtd          raid-check     sysstat
crond            libvirt-guests    rdisc          sysstat.ioconf
ebtables-config  man-db            readonly-root  virtlockd
fcoe             modules           rpcbind        wpa_supplicant
firewalld        netconsole        rsyncd
grub             network           rsyslog
init             network-scripts   run-parts
```

查阅资料后得知，CentOS 7使用`firewall`防火墙代替`iptables`防火墙，使用如下命令开放`8080`端口：

```
[root@localhost /]# firewall-cmd --permanent --zone=public --add-port=8080/tcp
success
```

更新防火墙规则，不重启服务，`firewall-cmd --reload`：

```
[root@localhost ~]# firewall-cmd --reload
success
```

查看`8080`端口是否开启，`firewall-cmd --query-port=8080/tcp`：

```
[root@localhost ~]# firewall-cmd --query-port=8080/tcp
yes
```

在本机之外的机器尝试访问tomcat：

```
http://192.168.80.128:8080/
```

### 删除暂存文件

* 删除暂存在`/usr/local/`路径下的`apache-tomcat-7.0.78.tar.gz`安装包：

```
[root@localhost local]# ls
apache-tomcat-7.0.78         bin  games    lib    libexec  share
apache-tomcat-7.0.78.tar.gz  etc  include  lib64  sbin     src
[root@localhost local]# rm -rf apache-tomcat-7.0.78.tar.gz 
[root@localhost local]# ls
apache-tomcat-7.0.78  etc    include  lib64    sbin   src
bin                   games  lib      libexec  share
```

* [参考：在CentOS中安装tomcat](https://jingyan.baidu.com/article/48a42057f140a4a9242504d4.html)
* [参考：CentOS中安装tomcat 8](http://www.linuxidc.com/Linux/2015-09/123118.htm)