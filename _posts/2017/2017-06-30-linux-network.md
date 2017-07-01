---
layout: post
title: "Linux网络管理"
categories: Linux
tags: Linux CentOS
author: Kopite
---

* content
{:toc}


Linux安装完之后并不能和网络中的其他机器进行通信，本文对Linux网络配置进行总结。



## 网络基础

### 端口

常见端口号：
* `FTP(文件传输协议)`：端口号 20 21
* `SSH(安全shell协议)`：端口号 22
* `telnet(远程登录协议)`：端口号 23
* `DNS(域名系统)`：端口号 53
* `http(超文本传输协议)`：端口号 80
* `SMTP(简单邮件传输协议)`：端口号 25
* `POP3(邮局协议3代)`：端口号 110

查看本机启用的端口：

```
[root@localhost ~]# netstat -an
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 ::1:631                 :::*                    LISTEN     
tcp6       0      0 ::1:25                  :::*                    LISTEN     
udp        0      0 192.168.80.128:45885    61.216.153.104:123      ESTABLISHED
udp        0      0 0.0.0.0:40852           0.0.0.0:*                          
udp        0      0 192.168.122.1:53        0.0.0.0:*                          
udp        0      0 0.0.0.0:67              0.0.0.0:*                          
udp        0      0 0.0.0.0:68              0.0.0.0:*                          
udp        0      0 0.0.0.0:5353            0.0.0.0:*                          
udp        0      0 0.0.0.0:51506           0.0.0.0:*                          
udp        0      0 127.0.0.1:323           0.0.0.0:*                          
udp        0      0 192.168.80.128:49497    94.237.64.20:123        ESTABLISHED
udp6       0      0 ::1:323                 :::*                               
udp6       0      0 :::35174                :::*                               
raw6       0      0 :::58                   :::*                    7          
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     10497    /run/systemd/journal/stdout
unix  2      [ ACC ]     STREAM     LISTENING     22401    @/tmp/dbus-sBGDB8hW
unix  7      [ ]         DGRAM                    10500    /run/systemd/journal/socket
unix  2      [ ACC ]     STREAM     LISTENING     16644    /var/run/cups/cups.sock
unix  24     [ ]         DGRAM                    10502    /dev/log
unix  2      [ ACC ]     STREAM     LISTENING     29940    /tmp/.esd-0/socket
unix  2      [ ACC ]     STREAM     LISTENING     22400    @/tmp/dbus-Gu6mCXHy
...
```

## Linux网络配置

### Linux配置IP地址

* `ifconfig`命令临时配置IP地址，例如，临时设置eth0网卡的IP地址与子网掩码：

```
ifconfig eth0 192.168.0.200 netmask 255.255.255.0
```

* setup工具永久配置IP地址：

```
[root@localhost ~]# setup
```

* 修改网络配置文件：
  1. 网卡信息文件，`/etc/sysconfig/network-scripts/ifcfg-网卡设备名`：
    * `DEVICE`，网卡设备名
    * `BOOTPROTO`，是否自动获取IP(none、static、dhcp)
    * `ONBOOT`，是否随网络服务启动
    * `TYPE="Ethernet"`，类型为以太网
```
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-eno16777736
```
```
TYPE="Ethernet"
BOOTPROTO="dhcp"
DEFROUTE="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_FAILURE_FATAL="no"
NAME="eno16777736"
UUID="50e81ed9-3bad-472e-aeeb-33a530664170"
DEVICE="eno16777736"
ONBOOT="yes"
```
  2. 主机名文件，`/etc/sysconfig/network`
```
[root@localhost ~]# vi /etc/sysconfig/network
```
  3. DNS配置文件，`/etc/resolv.conf`
```
[root@localhost ~]# vi /etc/resolv.conf
```

* 图形界面配置IP地址

### 虚拟机网络参数配置

1. 配置Linux IP地址：
```
[root@localhost ~]# setup
```
2. 修改网卡配置文件，`/etc/sysconfig/network-scripts/ifcfg-网卡设备名` -> `ONBOOT="yes"`：
```
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-eno16777736
```
3. 重启网络服务，`service network restart`：
```
[root@localhost ~]# service network restart 
```
4. UUID相同时，需要修改UUID（如果是自行安装的Linux系统，不需执行此步骤）：
  *  删除MAC地址行，`/etc/sysconfig/network-scripts/ifcfg-网卡设备名`
  *  删除网卡和MAC地址绑定文件，`/etc/udev/rules.d/70-persistent-net.rules`
  *  重新启动系统

### Linux网络命令

#### 网络测试命令

* `ping [选项] IP或域名`，探测指定IP或域名的网络状况，选项：
  * `-c 次数`，指定ping包的次数

* `telnet [域名或IP] [端口]`，远程管理与端口探测命令

* `traceroute [选项] IP或域名`，路由跟踪命令，选项：
  * `-n`，使用IP，不使用域名，速度更快

* `wget 下载地址`，下载命令

* [参考：慕课网](http://www.imooc.com/course/list?c=linux)