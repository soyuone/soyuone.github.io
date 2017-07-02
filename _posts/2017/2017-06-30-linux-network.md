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

### 常用端口

常用端口号：
* `FTP(文件传输协议)`：端口号 20 21
* `SSH(安全shell协议)`：端口号 22
* `telnet(远程登录协议)`：端口号 23
* `DNS(域名系统)`：端口号 53
* `http(超文本传输协议)`：端口号 80
* `SMTP(简单邮件传输协议)`：端口号 25
* `POP3(邮局协议3代)`：端口号 110

### firewall防火墙

CentOS 7使用`firewall`防火墙代替`iptables`防火墙。

#### 配置

* 系统配置目录，`/usr/lib/firewalld/services/`，用于存放定义的网络服务和端口参数，系统参数，不能修改:

```
[root@localhost services]# ls
amanda-client.xml        iscsi-target.xml  pop3s.xml
bacula-client.xml        kerberos.xml      postgresql.xml
bacula.xml               kpasswd.xml       proxy-dhcp.xml
dhcpv6-client.xml        ldaps.xml         radius.xml
dhcpv6.xml               ldap.xml          RH-Satellite-6.xml
dhcp.xml                 libvirt-tls.xml   rpc-bind.xml
dns.xml                  libvirt.xml       rsyncd.xml
freeipa-ldaps.xml        mdns.xml          samba-client.xml
freeipa-ldap.xml         mountd.xml        samba.xml
freeipa-replication.xml  ms-wbt.xml        smtp.xml
ftp.xml                  mysql.xml         ssh.xml
high-availability.xml    nfs.xml           telnet.xml
https.xml                ntp.xml           tftp-client.xml
http.xml                 openvpn.xml       tftp.xml
imaps.xml                pmcd.xml          transmission-client.xml
ipp-client.xml           pmproxy.xml       vdsm.xml
ipp.xml                  pmwebapis.xml     vnc-server.xml
ipsec.xml                pmwebapi.xml      wbem-https.xml
```

* 用户配置目录，`/etc/firewalld/`：

```
[root@localhost firewalld]# ls
firewalld.conf  icmptypes  lockdown-whitelist.xml  services  zones
```

#### 命令

##### 帮助文档

* 查看帮助文档，`firewall-cmd --help`：

```
[root@localhost ~]# firewall-cmd --help

Usage: firewall-cmd [OPTIONS...]

General Options
  -h, --help           Prints a short help text and exists
  -V, --version        Print the version string of firewalld
  -q, --quiet          Do not print status messages

Status Options
  --state              Return and print firewalld state
  --reload             Reload firewall and keep state information
  --complete-reload    Reload firewall and loose state information
  --runtime-to-permanent
                       Create permanent from runtime configuration

Permanent Options
  --permanent          Set an option permanently
                       Usable for options maked with [P]

Zone Options
  --get-default-zone   Print default zone for connections and interfaces
  --set-default-zone=<zone>
                       Set default zone
  --get-active-zones   Print currently active zones
  --get-zones          Print predefined zones [P]
  --get-services       Print predefined services [P]
  --get-icmptypes      Print predefined icmptypes [P]
  --get-zone-of-interface=<interface>
                       Print name of the zone the interface is bound to [P]
  --get-zone-of-source=<source>[/<mask>]
                       Print name of the zone the source[/mask] is bound to [P]
  --list-all-zones     List everything added for or enabled in all zones [P]
  --new-zone=<zone>    Add a new zone [P only]
  --delete-zone=<zone> Delete an existing zone [P only]
  --zone=<zone>        Use this zone to set or query options, else default zone
                       Usable for options maked with [Z]
  --get-target         Get the zone target [P] [Z]
  --set-target=<target>
                       Set the zone target [P] [Z]

IcmpType Options
  --new-icmptype=<icmptype>
                       Add a new icmptype [P only]
  --delete-icmptype=<icmptype>
                       Delete and existing icmptype [P only]

Service Options
  --new-service=<service>
                       Add a new service [P only]
  --delete-service=<service>
                       Delete and existing service [P only]
...
```

##### zone

* 查看已被激活的zone信息，`firewall-cmd --get-active-zones`
* 查看指定接口的zone信息，`firewall-cmd --get-zone-of-interface=网卡设备名`

```
[root@localhost ~]# firewall-cmd --get-active-zones
public
  interfaces: eno16777736
```

```
[root@localhost ~]# firewall-cmd --get-zone-of-interface=eno16777736
public
```

##### 运行状态

* 查看firewall运行状态，`firewall-cmd --state`：

```
[root@localhost ~]# firewall-cmd --state
running
```

##### 服务状态

* 查看firewalld服务状态，`systemctl status firewalld.service`：

```
[root@localhost ~]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 六 2017-07-01 21:07:41 CST; 13min ago
 Main PID: 4191 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─4191 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

7月 01 21:07:42 localhost.localdomain firewalld[4191]: 2017-07-01 21:07:42 E...
7月 01 21:07:42 localhost.localdomain firewalld[4191]: 2017-07-01 21:07:42 E...
7月 01 21:07:42 localhost.localdomain firewalld[4191]: 2017-07-01 21:07:42 E...
7月 01 21:07:42 localhost.localdomain firewalld[4191]: 2017-07-01 21:07:42 E...
7月 01 21:07:42 localhost.localdomain firewalld[4191]: 2017-07-01 21:07:42 E...
7月 01 21:07:42 localhost.localdomain firewalld[4191]: 2017-07-01 21:07:42 E...
7月 01 21:07:42 localhost.localdomain firewalld[4191]: 2017-07-01 21:07:42 E...
7月 01 21:07:42 localhost.localdomain firewalld[4191]: 2017-07-01 21:07:42 E...
7月 01 21:07:42 localhost.localdomain firewalld[4191]: 2017-07-01 21:07:42 E...
7月 01 21:08:48 localhost.localdomain systemd[1]: Started firewalld - dynami...
Hint: Some lines were ellipsized, use -l to show in full.
```

##### 防火墙规则

* 查看防火墙规则，`firewall-cmd --list-all`：

```
[root@localhost ~]# firewall-cmd --list-all
public (default, active)
  interfaces: eno16777736
  sources: 
  services: dhcpv6-client ssh
  ports: 
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules:
```

##### 开启

* 开启firewalld服务，`service firewalld start`/`systemctl start firewalld.service`：

```
[root@localhost ~]# service firewalld start
Redirecting to /bin/systemctl start  firewalld.service
```

```
[root@localhost ~]# systemctl start firewalld.service
```

##### 关闭

* 关闭firewalld服务，`service firewalld stop`/`systemctl stop firewalld.service`：

```
[root@localhost ~]# service firewalld stop
Redirecting to /bin/systemctl stop  firewalld.service
```

```
[root@localhost ~]# systemctl stop firewalld.service
```

##### 重启

* 重启firewalld服务，`service firewalld restart`/`systemctl restart firewalld.service`：

```
[root@localhost ~]# service firewalld restart
Redirecting to /bin/systemctl restart  firewalld.service
```

```
[root@localhost ~]# systemctl restart firewalld.service
```

##### 开机启动

* 开机启动，`systemctl enable firewalld`
* 取消开机启动，`systemctl disable firewalld`

```
[root@localhost ~]# systemctl enable firewalld
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/basic.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
```

```
[root@localhost ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
Removed symlink /etc/systemd/system/basic.target.wants/firewalld.service.
```

##### 开放端口

* 开放端口，`firewall-cmd --permanent --zone=public --add-port=端口号/通信协议`，选项：
  * `firwall-cmd`，Linux提供的操作firewall的工具
  * `--permanent`，表示设置永久生效，没有此参数将在重启后失效
  * `--zone=public`，作用域为public
  * `--add-port=端口号/通信协议`，添加端口

```
[root@localhost /]# firewall-cmd --permanent --zone=public --add-port=8080/tcp
success
```

##### 更新规则

* 更新防火墙规则，不重启服务，`firewall-cmd --reload`
* 更新防火墙规则，重启服务，`firewall-cmd --complete-reload`

```
[root@localhost ~]# firewall-cmd --reload
success
```

```
[root@localhost ~]# firewall-cmd --complete-reload
success
```

##### 移除端口

* 移除端口，`firewall-cmd --permanent --zone=public --remove-port=端口号/通信协议`，选项：
  * `--remove-port=端口号/通信协议`，移除端口

```
[root@localhost /]# firewall-cmd --permanent --zone=public --remove-port=8080/tcp
success
```

##### 端口状态

* 查看端口号是否开启，`firewall-cmd --query-port=端口号/通信协议`：

```
[root@localhost ~]# firewall-cmd --query-port=80/tcp
yes
[root@localhost ~]# firewall-cmd --query-port=8080/tcp
yes
```

## Linux网络配置

### 配置IP地址

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

### 网络命令

* `ping [选项] IP或域名`，探测指定IP或域名的网络状况，选项：
  * `-c 次数`，指定ping包的次数

* `telnet [域名或IP] [端口]`，远程管理与端口探测命令

* `traceroute [选项] IP或域名`，路由跟踪命令，选项：
  * `-n`，使用IP，不使用域名，速度更快

* `wget 下载地址`，下载命令

* [参考：慕课网](http://www.imooc.com/course/list?c=linux)
* [参考：CentOS 7中firewall防火墙详解和配置以及切换为iptables防火墙](http://blog.csdn.net/xlgen157387/article/details/52672988)
* [参考：CentOS 7下使用firewall](http://www.centoscn.com/image-text/config/2015/0730/5943.html)
* [参考：CentOS 7 firewall-cmd查看端口是否开放及开放端口](http://blog.csdn.net/y534560449/article/details/65629697)