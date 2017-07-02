---
layout: post
title: "mysql在Linux平台的安装和配置"
categories: database
tags: database mysql Linux CentOS
author: Kopite
---

* content
{:toc}


mysql`版本号5.6.36 MySQL Community Server (GPL)`在Linux平台的安装配置。



## 安装

### 准备工作

* 搜索服务器上mysql相关的包，`yum search mysql`，CentOS 7将mysql数据库从默认的程序列表中移除，改用mariadb代替mysql：

```
[root@localhost ~]# yum search mysql
已加载插件：fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
============================== N/S matched: mysql ==============================
MySQL-python.x86_64 : An interface to MySQL
akonadi-mysql.x86_64 : Akonadi MySQL backend support
apr-util-mysql.x86_64 : APR utility library MySQL DBD driver
dovecot-mysql.x86_64 : MySQL back end for dovecot
freeradius-mysql.x86_64 : MySQL support for freeradius
libdbi-dbd-mysql.x86_64 : MySQL plugin for libdbi
mysql-connector-java.noarch : Official JDBC driver for MySQL
mysql-connector-odbc.x86_64 : ODBC driver for MySQL
pcp-pmda-mysql.x86_64 : Performance Co-Pilot (PCP) metrics for MySQL
perl-DBD-MySQL.x86_64 : A MySQL interface for Perl
php-mysql.x86_64 : A module for PHP applications that use MySQL databases
php-mysqlnd.x86_64 : A module for PHP applications that use MySQL databases
qt-mysql.i686 : MySQL driver for Qt's SQL classes
qt-mysql.x86_64 : MySQL driver for Qt's SQL classes
qt3-MySQL.i686 : MySQL drivers for Qt 3's SQL classes
qt3-MySQL.x86_64 : MySQL drivers for Qt 3's SQL classes
qt5-qtbase-mysql.i686 : MySQL driver for Qt5's SQL classes
qt5-qtbase-mysql.x86_64 : MySQL driver for Qt5's SQL classes
redland-mysql.x86_64 : MySQL storage support for Redland
rsyslog-mysql.x86_64 : MySQL support for rsyslog
mariadb.x86_64 : A community developed branch of MySQL
mariadb-devel.i686 : Files for development of MariaDB/MySQL applications
mariadb-devel.x86_64 : Files for development of MariaDB/MySQL applications
mariadb-libs.x86_64 : The shared libraries required for MariaDB/MySQL clients
mariadb-libs.i686 : The shared libraries required for MariaDB/MySQL clients

  名称和简介匹配 only，使用“search all”试试。
```

* 从[mysql官网yum仓库](https://dev.mysql.com/downloads/repo/yum/)下载rpm包，如下所示：

![](/image/2017/2017-07-02-database-mysql-linux-configure-1.png)

* 使用[Bitvise SSH Client](https://www.bitvise.com/ssh-client)工具将下载的`mysql57-community-release-el7-11.noarch.rpm`包复制到`/usr/local/`路径，复制完毕后查看该路径下是否有文件：

```
[root@localhost local]# ls -l
总用量 32
drwxr-xr-x. 9 root root  4096 6月  30 21:09 apache-tomcat-7.0.78
drwxr-xr-x. 2 root root     6 8月  12 2015 bin
drwxr-xr-x. 2 root root     6 8月  12 2015 etc
drwxr-xr-x. 2 root root     6 8月  12 2015 games
drwxr-xr-x. 2 root root     6 8月  12 2015 include
drwxr-xr-x. 2 root root     6 8月  12 2015 lib
drwxr-xr-x. 2 root root     6 8月  12 2015 lib64
drwxr-xr-x. 2 root root     6 8月  12 2015 libexec
-rw-r--r--. 1 root root 25680 7月   2 11:10 mysql57-community-release-el7-11.noarch.rpm
drwxr-xr-x. 2 root root     6 8月  12 2015 sbin
drwxr-xr-x. 5 root root    46 6月  20 19:37 share
drwxr-xr-x. 2 root root     6 8月  12
```

* 通过yum方式来安装本地rpm包，`yum localinstall mysql57-community-release-el7-11.noarch.rpm`：

```
[root@localhost local]# yum localinstall mysql57-community-release-el7-11.noarch.rpm
已加载插件：fastestmirror, langpacks
正在检查 mysql57-community-release-el7-11.noarch.rpm: mysql57-community-release-el7-11.noarch
mysql57-community-release-el7-11.noarch.rpm 将被安装
正在解决依赖关系
--> 正在检查事务
---> 软件包 mysql57-community-release.noarch.0.el7-11 将被 安装
--> 解决依赖关系完成
base/7/x86_64                                            | 3.6 kB     00:00     
extras/7/x86_64                                          | 3.4 kB     00:00     
updates/7/x86_64                                         | 3.4 kB     00:00     

依赖关系解决

================================================================================
 Package           架构   版本   源                                        大小
================================================================================
正在安装:
 mysql57-community-release
                   noarch el7-11 /mysql57-community-release-el7-11.noarch  31 k

事务概要
================================================================================
安装  1 软件包

总计：31 k
安装大小：31 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
警告：RPM 数据库已被非 yum 程序修改。
  正在安装    : mysql57-community-release-el7-11.noarch                     1/1 
  验证中      : mysql57-community-release-el7-11.noarch                     1/1 

已安装:
  mysql57-community-release.noarch 0:el7-11                                     

完毕！
```

* 查看mysql源是否安装成功，`yum repolist all | grep mysql`：

```
[root@localhost ~]# yum repolist all | grep mysql
mysql-cluster-7.5-community/x86_64 MySQL Cluster 7.5 Community       禁用
mysql-cluster-7.5-community-source MySQL Cluster 7.5 Community - Sou 禁用
mysql-cluster-7.6-community/x86_64 MySQL Cluster 7.6 Community       禁用
mysql-cluster-7.6-community-source MySQL Cluster 7.6 Community - Sou 禁用
mysql-connectors-community/x86_64  MySQL Connectors Community        启用:    36
mysql-connectors-community-source  MySQL Connectors Community - Sour 禁用
mysql-tools-community/x86_64       MySQL Tools Community             启用:    47
mysql-tools-community-source       MySQL Tools Community - Source    禁用
mysql-tools-preview/x86_64         MySQL Tools Preview               禁用
mysql-tools-preview-source         MySQL Tools Preview - Source      禁用
mysql55-community/x86_64           MySQL 5.5 Community Server        禁用
mysql55-community-source           MySQL 5.5 Community Server - Sour 禁用
mysql56-community/x86_64           MySQL 5.6 Community Server        禁用
mysql56-community-source           MySQL 5.6 Community Server - Sour 禁用
mysql57-community/x86_64           MySQL 5.7 Community Server        启用:   187
mysql57-community-source           MySQL 5.7 Community Server - Sour 禁用
mysql80-community/x86_64           MySQL 8.0 Community Server        禁用
mysql80-community-source           MySQL 8.0 Community Server - Sour 禁用
```

### 选择版本

* 可通过修改`/etc/yum.repos.d/mysql-community.repo`，选择安装的mysql版本。例如，安装mysql 5.6版本时，将`[mysql57-community]`中的`enabled=1`改成`enabled=0`，再将`[mysql56-community]`中的`enabled=0`改成`enabled=1`，如下所示：

```
[root@localhost yum.repos.d]# vi mysql-community.repo
```

```
# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

```
[root@localhost yum.repos.d]# yum repolist all | grep mysql
mysql-cluster-7.5-community/x86_64 MySQL Cluster 7.5 Community       禁用
mysql-cluster-7.5-community-source MySQL Cluster 7.5 Community - Sou 禁用
mysql-cluster-7.6-community/x86_64 MySQL Cluster 7.6 Community       禁用
mysql-cluster-7.6-community-source MySQL Cluster 7.6 Community - Sou 禁用
mysql-connectors-community/x86_64  MySQL Connectors Community        启用:    36
mysql-connectors-community-source  MySQL Connectors Community - Sour 禁用
mysql-tools-community/x86_64       MySQL Tools Community             启用:    47
mysql-tools-community-source       MySQL Tools Community - Source    禁用
mysql-tools-preview/x86_64         MySQL Tools Preview               禁用
mysql-tools-preview-source         MySQL Tools Preview - Source      禁用
mysql55-community/x86_64           MySQL 5.5 Community Server        禁用
mysql55-community-source           MySQL 5.5 Community Server - Sour 禁用
mysql56-community/x86_64           MySQL 5.6 Community Server        启用:   327
mysql56-community-source           MySQL 5.6 Community Server - Sour 禁用
mysql57-community/x86_64           MySQL 5.7 Community Server        禁用
mysql57-community-source           MySQL 5.7 Community Server - Sour 禁用
mysql80-community/x86_64           MySQL 8.0 Community Server        禁用
mysql80-community-source           MySQL 8.0 Community Server - Sour 禁用
```

### 安装

* 安装mysql，`yum install mysql-community-server`：

```
[root@localhost ~]# yum install mysql-community-server
已加载插件：fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirrors.hust.edu.cn
 * updates: mirror.bit.edu.cn
正在解决依赖关系
--> 正在检查事务
---> 软件包 mysql-community-server.x86_64.0.5.6.36-2.el7 将被 安装
--> 正在处理依赖关系 mysql-community-common(x86-64) = 5.6.36-2.el7，它被软件包 mysql-community-server-5.6.36-2.el7.x86_64 需要
--> 正在处理依赖关系 mysql-community-client(x86-64) >= 5.6.10，它被软件包 mysql-community-server-5.6.36-2.el7.x86_64 需要
--> 正在处理依赖关系 perl(DBI)，它被软件包 mysql-community-server-5.6.36-2.el7.x86_64 需要
--> 正在检查事务
---> 软件包 mysql-community-client.x86_64.0.5.6.36-2.el7 将被 安装
--> 正在处理依赖关系 mysql-community-libs(x86-64) >= 5.6.10，它被软件包 mysql-community-client-5.6.36-2.el7.x86_64 需要
---> 软件包 mysql-community-common.x86_64.0.5.6.36-2.el7 将被 安装
---> 软件包 perl-DBI.x86_64.0.1.627-4.el7 将被 安装
--> 正在处理依赖关系 perl(RPC::PlServer) >= 0.2001，它被软件包 perl-DBI-1.627-4.el7.x86_64 需要
--> 正在处理依赖关系 perl(RPC::PlClient) >= 0.2000，它被软件包 perl-DBI-1.627-4.el7.x86_64 需要
--> 正在检查事务
---> 软件包 mariadb-libs.x86_64.1.5.5.44-2.el7.centos 将被 取代
---> 软件包 mysql-community-libs.x86_64.0.5.6.36-2.el7 将被 舍弃
---> 软件包 perl-PlRPC.noarch.0.0.2020-14.el7 将被 安装
--> 正在处理依赖关系 perl(Net::Daemon) >= 0.13，它被软件包 perl-PlRPC-0.2020-14.el7.noarch 需要
--> 正在处理依赖关系 perl(Net::Daemon::Test)，它被软件包 perl-PlRPC-0.2020-14.el7.noarch 需要
--> 正在处理依赖关系 perl(Net::Daemon::Log)，它被软件包 perl-PlRPC-0.2020-14.el7.noarch 需要
--> 正在处理依赖关系 perl(Compress::Zlib)，它被软件包 perl-PlRPC-0.2020-14.el7.noarch 需要
--> 正在检查事务
---> 软件包 perl-IO-Compress.noarch.0.2.061-2.el7 将被 安装
--> 正在处理依赖关系 perl(Compress::Raw::Zlib) >= 2.061，它被软件包 perl-IO-Compress-2.061-2.el7.noarch 需要
--> 正在处理依赖关系 perl(Compress::Raw::Bzip2) >= 2.061，它被软件包 perl-IO-Compress-2.061-2.el7.noarch 需要
---> 软件包 perl-Net-Daemon.noarch.0.0.48-5.el7 将被 安装
--> 正在检查事务
---> 软件包 perl-Compress-Raw-Bzip2.x86_64.0.2.061-3.el7 将被 安装
---> 软件包 perl-Compress-Raw-Zlib.x86_64.1.2.061-4.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package                   架构     版本              源                   大小
================================================================================
正在安装:
 mysql-community-libs      x86_64   5.6.36-2.el7      mysql56-community   2.0 M
      替换  mariadb-libs.x86_64 1:5.5.44-2.el7.centos
 mysql-community-server    x86_64   5.6.36-2.el7      mysql56-community    59 M
为依赖而安装:
 mysql-community-client    x86_64   5.6.36-2.el7      mysql56-community    19 M
 mysql-community-common    x86_64   5.6.36-2.el7      mysql56-community   257 k
 perl-Compress-Raw-Bzip2   x86_64   2.061-3.el7       base                 32 k
 perl-Compress-Raw-Zlib    x86_64   1:2.061-4.el7     base                 57 k
 perl-DBI                  x86_64   1.627-4.el7       base                802 k
 perl-IO-Compress          noarch   2.061-2.el7       base                260 k
 perl-Net-Daemon           noarch   0.48-5.el7        base                 51 k
 perl-PlRPC                noarch   0.2020-14.el7     base                 36 k

事务概要
================================================================================
安装  2 软件包 (+8 依赖软件包)

总计：82 M
总下载量：79 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for mysql56-community
警告：/var/cache/yum/x86_64/7/mysql56-community/packages/mysql-community-client-5.6.36-2.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
mysql-community-client-5.6.36-2.el7.x86_64.rpm 的公钥尚未安装
(1/2): mysql-community-client-5.6.36-2.el7.x86_64.rpm      |  19 MB   00:31     
mysql-community-server-5.6.36- FAILED                                           
http://repo.mysql.com/yum/mysql-5.6-community/el/7/x86_64/mysql-community-server-5.6.36-2.el7.x86_64.rpm: [Errno 12] Timeout on http://repo.mysql.com/yum/mysql-5.6-community/el/7/x86_64/mysql-community-server-5.6.36-2.el7.x86_64.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
正在尝试其它镜像。
(2/2): mysql-community-server-5.6.36-2.el7.x86_64.rpm      |  59 MB   00:43     
--------------------------------------------------------------------------------
总计                                               377 kB/s |  79 MB  03:33     
从 file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql 检索密钥
导入 GPG key 0x5072E1F5:
 用户ID     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 指纹       : a4a9 4068 76fc bd3c 4567 70c8 8c71 8d3b 5072 e1f5
 软件包     : mysql57-community-release-el7-11.noarch (@/mysql57-community-release-el7-11.noarch)
 来自       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
是否继续？[y/N]：y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : mysql-community-common-5.6.36-2.el7.x86_64                 1/11 
  正在安装    : mysql-community-libs-5.6.36-2.el7.x86_64                   2/11 
  正在安装    : mysql-community-client-5.6.36-2.el7.x86_64                 3/11 
  正在安装    : perl-Net-Daemon-0.48-5.el7.noarch                          4/11 
  正在安装    : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                 5/11 
  正在安装    : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                6/11 
  正在安装    : perl-IO-Compress-2.061-2.el7.noarch                        7/11 
  正在安装    : perl-PlRPC-0.2020-14.el7.noarch                            8/11 
  正在安装    : perl-DBI-1.627-4.el7.x86_64                                9/11 
  正在安装    : mysql-community-server-5.6.36-2.el7.x86_64                10/11 
  正在删除    : 1:mariadb-libs-5.5.44-2.el7.centos.x86_64                 11/11 
  验证中      : mysql-community-common-5.6.36-2.el7.x86_64                 1/11 
  验证中      : mysql-community-server-5.6.36-2.el7.x86_64                 2/11 
  验证中      : mysql-community-client-5.6.36-2.el7.x86_64                 3/11 
  验证中      : perl-PlRPC-0.2020-14.el7.noarch                            4/11 
  验证中      : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                5/11 
  验证中      : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                 6/11 
  验证中      : perl-Net-Daemon-0.48-5.el7.noarch                          7/11 
  验证中      : perl-DBI-1.627-4.el7.x86_64                                8/11 
  验证中      : perl-IO-Compress-2.061-2.el7.noarch                        9/11 
  验证中      : mysql-community-libs-5.6.36-2.el7.x86_64                  10/11 
  验证中      : 1:mariadb-libs-5.5.44-2.el7.centos.x86_64                 11/11 

已安装:
  mysql-community-libs.x86_64 0:5.6.36-2.el7                                    
  mysql-community-server.x86_64 0:5.6.36-2.el7                                  

作为依赖被安装:
  mysql-community-client.x86_64 0:5.6.36-2.el7                                  
  mysql-community-common.x86_64 0:5.6.36-2.el7                                  
  perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7                                  
  perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7                                   
  perl-DBI.x86_64 0:1.627-4.el7                                                 
  perl-IO-Compress.noarch 0:2.061-2.el7                                         
  perl-Net-Daemon.noarch 0:0.48-5.el7                                           
  perl-PlRPC.noarch 0:0.2020-14.el7                                             

替代:
  mariadb-libs.x86_64 1:5.5.44-2.el7.centos                                     

完毕！
```

### 启动服务

* 启动mysql服务，`systemctl start mysqld`：

```
[root@localhost ~]# systemctl start mysqld
```

### 停止服务

* 停止mysql服务，`systemctl stop mysqld`：

```
[root@localhost ~]# systemctl stop mysqld
```

### 服务状态

* 查看mysql服务启动状态，`systemctl status mysqld`：

```
[root@localhost ~]# systemctl status mysqld
● mysqld.service - MySQL Community Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2017-07-02 12:32:21 CST; 17min ago
  Process: 6350 ExecStartPost=/usr/bin/mysql-systemd-start post (code=exited, status=0/SUCCESS)
  Process: 6279 ExecStartPre=/usr/bin/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
 Main PID: 6349 (mysqld_safe)
   CGroup: /system.slice/mysqld.service
           ├─6349 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─6516 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --...

7月 02 12:32:18 localhost.localdomain mysql-systemd-start[6279]: 2017-07-02 ...
7月 02 12:32:20 localhost.localdomain mysql-systemd-start[6279]: 2017-07-02 ...
7月 02 12:32:20 localhost.localdomain mysql-systemd-start[6279]: PLEASE REME...
7月 02 12:32:20 localhost.localdomain mysql-systemd-start[6279]: To do so, s...
7月 02 12:32:20 localhost.localdomain mysql-systemd-start[6279]: /usr/bin/my...
7月 02 12:32:20 localhost.localdomain mysql-systemd-start[6279]: /usr/bin/my...
7月 02 12:32:20 localhost.localdomain mysql-systemd-start[6279]: Alternative...
7月 02 12:32:20 localhost.localdomain mysqld_safe[6349]: 170702 12:32:20 mys...
7月 02 12:32:20 localhost.localdomain mysqld_safe[6349]: 170702 12:32:20 mys...
7月 02 12:32:21 localhost.localdomain systemd[1]: Started MySQL Community Se...
Hint: Some lines were ellipsized, use -l to show in full.
```

### 开机启动服务

* 设置开机自动启动mysql服务，`systemctl enable mysqld`
* 取消开机自动启动mysql服务，`systemctl disable mysqld`
* 重新加载配置，`systemctl daemon-reload`

```
[root@localhost ~]# systemctl enable mysqld
Created symlink from /etc/systemd/system/mysql.service to /usr/lib/systemd/system/mysqld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/mysqld.service to /usr/lib/systemd/system/mysqld.service.
```

```
[root@localhost ~]# systemctl disable mysqld
Removed symlink /etc/systemd/system/multi-user.target.wants/mysqld.service.
Removed symlink /etc/systemd/system/mysql.service.
```

```
[root@localhost ~]# systemctl daemon-reload
```

### 登录密码

* 安装后的mysql按回车键即可登录，修改mysql登录密码，`SET PASSWORD FOR 'root'@'localhost' = PASSWORD('admin');`：

```
[root@localhost ~]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.36 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('admin');
Query OK, 0 rows affected (0.00 sec)
```

### 字符集

* 修改`/etc/my.cnf`，配置默认字符集，字符集名称必须与`/usr/share/mysql/charsets/Index.xml`中一致，如下所示：

```
...
<charset name="utf8">
  <family>Unicode</family>
  <description>UTF-8 Unicode</description>
  <alias>utf-8</alias>
  <collation name="utf8_general_ci"     id="33">
   <flag>primary</flag>
   <flag>compiled</flag>
  </collation>
  <collation name="utf8_bin"            id="83">
    <flag>binary</flag>
    <flag>compiled</flag>
  </collation>
</charset>
...
```

```
[root@localhost etc]# vi my.cnf
```

```
[mysql]
default-character-set=utf8
```

### 远程连接

* 远程连接设置，把在所有数据库的所有表的所有权限赋值给位于所有IP地址的root用户，`grant all privileges on *.* to root@'%'identified by 'mysql登录密码';`：

```
mysql> grant all privileges on *.* to root@'%'identified by 'admin';
Query OK, 0 rows affected (0.01 sec)
```

### 3306端口

* 防火墙开放3306端口：

```
[root@localhost etc]# firewall-cmd --permanent --zone=public --add-port=3306/tcp 
success
```

```
[root@localhost etc]# firewall-cmd --reload
success
```

```
[root@localhost etc]# firewall-cmd --query-port=3306/tcp
yes
```

### 删除暂存文件

* 删除暂存在`/usr/local/`路径下的`mysql57-community-release-el7-11.noarch.rpm`：

```
[root@localhost local]# ls
apache-tomcat-7.0.78  include  mysql57-community-release-el7-11.noarch.rpm
bin                   lib      sbin
etc                   lib64    share
games                 libexec  src
[root@localhost local]# rm -rf mysql57-community-release-el7-11.noarch.rpm 
[root@localhost local]# ls
apache-tomcat-7.0.78  etc    include  lib64    sbin   src
bin                   games  lib      libexec  share
```

* [参考：mysql 5.7安装与配置](http://blog.csdn.net/xyang81/article/details/51759200)
* [参考：CentOS 7中mysql数据库的安装和配置](http://www.cnblogs.com/starof/p/4680083.html)