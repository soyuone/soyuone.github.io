---
layout: post
title: "Linux网络管理"
categories: Linux
tags: Linux CentOS
author: Kopite
---

* content
{:toc}


Linux装好以后并不能和网络中的其他机器进行通信，本文对Linux网络配置进行总结。



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
unix  2      [ ACC ]     STREAM     LISTENING     16656    /var/run/rpcbind.sock
unix  2      [ ACC ]     STREAM     LISTENING     32790    /run/user/0/at-spi2-socket-3461
unix  2      [ ACC ]     STREAM     LISTENING     29171    @/tmp/.ICE-unix/3130
unix  2      [ ACC ]     STREAM     LISTENING     31002    /run/user/0/at-spi2-socket-3441
unix  2      [ ACC ]     STREAM     LISTENING     24639    private/tlsmgr
unix  2      [ ACC ]     STREAM     LISTENING     16663    /var/run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     29213    /run/user/0/keyring/pkcs11
unix  2      [ ACC ]     STREAM     LISTENING     24731    @/tmp/.X11-unix/X0
unix  2      [ ACC ]     STREAM     LISTENING     24648    private/defer
```

* [参考：慕课网](http://www.imooc.com/course/list?c=linux)