---
layout: post
title: "Linux软件安装管理"
categories: Linux
tags: Linux CentOS
author: Kopite
---

* content
{:toc}


Windows中的软件在Linux中无法安装。
<br>
<br>
生产环境下的Linux服务器通常没有图形用户界面，需要在字符界面下安装软件。



## 概述

Linux中的软件包分为`源码包`和`二进制包（rpm包、系统默认包）`。

## rpm命令管理

rpm包在Linux系统光盘的`/mnt/cdrom/Packages`目录中，因此需要挂载光盘，光盘挂载完成后进入`/mnt/cdrom/Packages`目录查看rpm包：

```
[root@localhost /]# cd /mnt/cdrom/
[root@localhost cdrom]# ls
CentOS_BuildTag  GPL       LiveOS    RPM-GPG-KEY-CentOS-7
EFI              images    Packages  RPM-GPG-KEY-CentOS-Testing-7
EULA             isolinux  repodata  TRANS.TBL
[root@localhost cdrom]# cd Packages/
[root@localhost Packages]# ls
389-ds-base-1.3.4.0-19.el7.x86_64.rpm
389-ds-base-libs-1.3.4.0-19.el7.x86_64.rpm
abattis-cantarell-fonts-0.0.16-3.el7.noarch.rpm
abrt-2.1.11-36.el7.centos.x86_64.rpm
...
zziplib-0.13.62-5.el7.x86_64.rpm
```

### 命名规则

rpm包的命名规则，以`abrt-2.1.11-36.el7.centos.x86_64.rpm`为例：
* abrt：软件报名
* 2.1.11：软件版本
* 36：软件发布的次数
* el7.centos：适合的Linux平台
* x86_64：适合的硬件平台
* rpm：rpm包扩展名

包全名与包名的关系，以`abrt-2.1.11-36.el7.centos.x86_64.rpm`为例：
* 包全名，操作的包是没有安装的软件包时，使用包全名，即`abrt-2.1.11-36.el7.centos.x86_64.rpm`，使用时要注意路径
* 包名，操作已经安装的软件包时，使用包名`abrt`，是搜索`/var/lib/rpm`中的数据库

### 依赖性

rpm包的依赖关系：
* 树形依赖：a -> b -> c
* 环形依赖：a -> b -> c -> a
* 模块依赖：[模块依赖查询网站](http://www.rpmfind.net/)

```
[root@localhost Packages]# rpm -ivh httpd-2.4.6-40.el7.centos.x86_64.rpm 
警告：httpd-2.4.6-40.el7.centos.x86_64.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY
错误：依赖检测失败：
	/etc/mime.types 被 httpd-2.4.6-40.el7.centos.x86_64 需要
	httpd-tools = 2.4.6-40.el7.centos 被 httpd-2.4.6-40.el7.centos.x86_64 需要
```

### 安装

* `rpm -ivh 包全名`，选项：
  * `-i`，安装
  * `-v`，显示详细信息
  * `-h`，显示进度

* rpm包默认的安装位置，不建议指定安装位置：
  * `/etc/`，配置文件安装目录
  * `/usr/bin/`，可执行的命令安装目录
  * `/usr/lib/`，程序所使用的函数库保存位置
  * `/usr/share/doc/`，基本的软件使用手册保存位置
  * `/usr/share/man/`，帮助文件保存位置

```
[root@localhost Packages]# rpm -ivh httpd-2.4.6-40.el7.centos.x86_64.rpm 
警告：httpd-2.4.6-40.el7.centos.x86_64.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY
错误：依赖检测失败：
	/etc/mime.types 被 httpd-2.4.6-40.el7.centos.x86_64 需要
	httpd-tools = 2.4.6-40.el7.centos 被 httpd-2.4.6-40.el7.centos.x86_64 需要
```

```
[root@localhost songyu]# rpm --help | grep prefix
  --prefix=<dir>                   如果可重定位，便把软件包重定位到 <dir>
```

### 升级

`rpm -Uvh 包全名`，选项：
* `-U`，升级

### 卸载

`rpm -e 包名`

### 查询

* `rpm -q 包名`，查询包是否安装，选项：
  * `-q`，查询

* `rpm -qa`，查询所有已经安装的rpm包，选项：
  * `-a`，所有

```
[root@localhost Packages]# rpm -qa | grep jdk
java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64
java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64
java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64
```

* `rpm -qi 包名`，查询软件包详细信息，选项：
  * `-i`，查询软件信息
  * `-p`，查询未安装的软件包信息

```
[root@localhost Packages]# rpm -qi java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64
Name        : java-1.7.0-openjdk
Epoch       : 1
Version     : 1.7.0.91
Release     : 2.6.2.3.el7
Architecture: x86_64
Install Date: 2017年06月20日 星期二 19时46分09秒
Group       : Development/Languages
Size        : 505037
License     : ASL 1.1 and ASL 2.0 and GPL+ and GPLv2 and GPLv2 with exceptions and LGPL+ and LGPLv2 and MPLv1.0 and MPLv1.1 and Public Domain and W3C
Signature   : RSA/SHA256, 2015年11月25日 星期三 22时45分07秒, Key ID 24c6a8a7f4a80eb5
Source RPM  : java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.src.rpm
Build Date  : 2015年11月21日 星期六 01时12分38秒
Build Host  : worker1.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://openjdk.java.net/
Summary     : OpenJDK Runtime Environment
Description :
The OpenJDK runtime environment.
```

```
[root@localhost Packages]# rpm -qip zenity-3.8.0-5.el7.x86_64.rpm
警告：zenity-3.8.0-5.el7.x86_64.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY
Name        : zenity
Version     : 3.8.0
Release     : 5.el7
Architecture: x86_64
Install Date: (not installed)
Group       : Applications/System
Size        : 5606592
License     : LGPLv2+
Signature   : RSA/SHA256, 2015年11月26日 星期四 00时07分26秒, Key ID 24c6a8a7f4a80eb5
Source RPM  : zenity-3.8.0-5.el7.src.rpm
Build Date  : 2015年11月20日 星期五 22时08分41秒
Build Host  : worker1.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://directory.fsf.org/zenity.html
Summary     : Display dialog boxes from shell scripts
Description :
Zenity lets you display Gtk+ dialog boxes from the command line and through
shell scripts. It is similar to gdialog, but is intended to be saner. It comes
from the same family as dialog, Xdialog, and cdialog.
```

* `rpm -ql 包名`，查询包中文件的安装位置，选项：
  * `-l`，列表
  * `-p`，查询未安装的软件包打算把包中文件安装的位置

```
[root@localhost Packages]# rpm -ql java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64/jre/lib/amd64/libjsoundalsa.so
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64/jre/lib/amd64/libpulse-java.so
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64/jre/lib/amd64/libsplashscreen.so
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64/jre/lib/amd64/xawt/libmawt.so
/usr/share/icons/hicolor/16x16/apps/java-1.7.0.png
/usr/share/icons/hicolor/24x24/apps/java-1.7.0.png
/usr/share/icons/hicolor/32x32/apps/java-1.7.0.png
/usr/share/icons/hicolor/48x48/apps/java-1.7.0.png
```

```
[root@localhost Packages]# rpm -qlp zenity-3.8.0-5.el7.x86_64.rpm
警告：zenity-3.8.0-5.el7.x86_64.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY
/usr/bin/zenity
/usr/share/doc/zenity-3.8.0
/usr/share/doc/zenity-3.8.0/AUTHORS
/usr/share/doc/zenity-3.8.0/COPYING
/usr/share/doc/zenity-3.8.0/NEWS
/usr/share/doc/zenity-3.8.0/README
/usr/share/doc/zenity-3.8.0/THANKS
/usr/share/help/C/zenity
/usr/share/help/C/zenity/calendar.page
/usr/share/help/C/zenity/color-selection.page
/usr/share/help/C/zenity/entry.page
/usr/share/help/C/zenity/error.page
/usr/share/help/C/zenity/figures
/usr/share/help/C/zenity/figures/zenity-calendar-screenshot.png
/usr/share/help/C/zenity/figures/zenity-colorselection-screenshot.png
/usr/share/help/C/zenity/figures/zenity-entry-screenshot.png
...
```

* `rpm -qf 文件名`，查询文件属于哪个rpm包，文件必须是通过rpm安装产生的文件，选项：
  * `-f`，查询文件属于哪个软件包

```
[root@localhost Packages]# rpm -qf /etc/yum.conf 
yum-3.4.3-132.el7.centos.0.1.noarch
```

* `rpm -qR 包名`，查询软件包的依赖性，选项：
  * `-R`，查询软件包的依赖性
  * `-p`，查询未安装的软件包的依赖性

```
[root@localhost Packages]# rpm -qR java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64 
/bin/sh
/bin/sh
/bin/sh
fontconfig
java-1.7.0-openjdk-headless = 1:1.7.0.91-2.6.2.3.el7
libX11.so.6()(64bit)
libXext.so.6()(64bit)
libXi.so.6()(64bit)
libXrender.so.1()(64bit)
libXtst.so.6()(64bit)
libasound.so.2()(64bit)
libasound.so.2(ALSA_0.9)(64bit)
...
```

```
[root@localhost Packages]# rpm -qRp zenity-3.8.0-5.el7.x86_64.rpm 
警告：zenity-3.8.0-5.el7.x86_64.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY
libX11.so.6()(64bit)
libatk-1.0.so.0()(64bit)
libc.so.6()(64bit)
libc.so.6(GLIBC_2.2.5)(64bit)
libc.so.6(GLIBC_2.3.4)(64bit)
libc.so.6(GLIBC_2.4)(64bit)
libcairo-gobject.so.2()(64bit)
libcairo.so.2()(64bit)
libgdk-3.so.0()(64bit)
libgdk_pixbuf-2.0.so.0()(64bit)
libgio-2.0.so.0()(64bit)
libglib-2.0.so.0()(64bit)
libgobject-2.0.so.0()(64bit)
libgtk-3.so.0()(64bit)
libnotify.so.4()(64bit)
...
```

### 校验

* `rpm -V 已安装的包名`，rpm包校验，有提示时表示包中文件被修改，选项：
  * `-V`，校验指定rpm包中的文件

* `rpm2cpio 包全名 | cpio -idv .文件绝对路径`，rpm包中文件提取

## yum在线安装

rpm包在安装过程中的依赖性较强，采用yum在线安装更便捷。`yum在线安装`，将所有软件包放在官方服务器上，进行在线安装时可以自动解决依赖性问题。

### yum源文件

查看yum源文件：

```
[root@localhost Packages]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# ls
CentOS-Base.repo  CentOS-Debuginfo.repo  CentOS-Media.repo    CentOS-Vault.repo
CentOS-CR.repo    CentOS-fasttrack.repo  CentOS-Sources.repo
[root@localhost yum.repos.d]# vi CentOS-Base.repo 
```

### yum命令

#### 查询

* `yum list`，查询服务器上所有可用软件包列表
* `yum search 关键字`，搜索服务器上所有和关键字相关的包

```
[root@localhost songyu]# yum search jdk
已加载插件：fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: ftp.sjtu.edu.cn
 * extras: mirrors.163.com
 * updates: centos.ustc.edu.cn
=============================== N/S matched: jdk ===============================
copy-jdk-configs.noarch : JDKs configuration files copier
java-1.6.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.6.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.6.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.6.0-openjdk-javadoc.x86_64 : OpenJDK API Documentation
java-1.6.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.7.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.7.0-openjdk-accessibility.x86_64 : OpenJDK accessibility connector
java-1.7.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.7.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.7.0-openjdk-headless.x86_64 : The OpenJDK runtime environment without
                                   : audio and video support
java-1.7.0-openjdk-javadoc.noarch : OpenJDK API Documentation
java-1.7.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.8.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk.i686 : OpenJDK Runtime Environment
java-1.8.0-openjdk-accessibility.x86_64 : OpenJDK accessibility connector
java-1.8.0-openjdk-accessibility-debug.x86_64 : OpenJDK accessibility connector
                                              : for packages with debug on
java-1.8.0-openjdk-debug.i686 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-debug.x86_64 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.8.0-openjdk-demo-debug.x86_64 : OpenJDK Demos with full debug on
java-1.8.0-openjdk-devel.i686 : OpenJDK Development Environment
java-1.8.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.8.0-openjdk-devel-debug.i686 : OpenJDK Development Environment with full
                                    : debug on
java-1.8.0-openjdk-devel-debug.x86_64 : OpenJDK Development Environment with
                                      : full debug on
java-1.8.0-openjdk-headless.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk-headless.i686 : OpenJDK Runtime Environment
java-1.8.0-openjdk-headless-debug.i686 : OpenJDK Runtime Environment with full
                                       : debug on
java-1.8.0-openjdk-headless-debug.x86_64 : OpenJDK Runtime Environment with full
                                         : debug on
java-1.8.0-openjdk-javadoc.noarch : OpenJDK API Documentation
java-1.8.0-openjdk-javadoc-debug.noarch : OpenJDK API Documentation for packages
                                        : with debug on
java-1.8.0-openjdk-javadoc-zip.noarch : OpenJDK API Documentation compressed in
                                      : single archive
java-1.8.0-openjdk-javadoc-zip-debug.noarch : OpenJDK API Documentation
     ...: compressed in single archive for packages with debug on
java-1.8.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.8.0-openjdk-src-debug.x86_64 : OpenJDK Source Bundle for packages with
                                    : debug on
ldapjdk-javadoc.noarch : Javadoc for ldapjdk
icedtea-web.x86_64 : Additional Java components for OpenJDK - Java browser
                   : plug-in and Web Start implementation
ldapjdk.noarch : The Mozilla LDAP Java SDK

  名称和简介匹配 only，使用“search all”试试。
```

* 查询仓库列表，`yum repolist all/enabled/disabled`：
  * `yum repolist all`，显示所有的仓库列表
  * `yum repolist enabled`，显示可用的仓库列表
  * `yum repolist disabled`，显示不可用的仓库列表

```
[root@localhost ~]# yum repolist all
已加载插件：fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirrors.hust.edu.cn
 * updates: mirror.bit.edu.cn
源标识                             源名称                            状态
C7.0.1406-base/x86_64              CentOS-7.0.1406 - Base            禁用
C7.0.1406-centosplus/x86_64        CentOS-7.0.1406 - CentOSPlus      禁用
C7.0.1406-extras/x86_64            CentOS-7.0.1406 - Extras          禁用
C7.0.1406-fasttrack/x86_64         CentOS-7.0.1406 - CentOSPlus      禁用
C7.0.1406-updates/x86_64           CentOS-7.0.1406 - Updates         禁用
C7.1.1503-base/x86_64              CentOS-7.1.1503 - Base            禁用
C7.1.1503-centosplus/x86_64        CentOS-7.1.1503 - CentOSPlus      禁用
C7.1.1503-extras/x86_64            CentOS-7.1.1503 - Extras          禁用
C7.1.1503-fasttrack/x86_64         CentOS-7.1.1503 - CentOSPlus      禁用
C7.1.1503-updates/x86_64           CentOS-7.1.1503 - Updates         禁用
base/7/x86_64                      CentOS-7 - Base                   启用: 9,363
base-debuginfo/x86_64              CentOS-7 - Debuginfo              禁用
base-source/7                      CentOS-7 - Base Sources           禁用
c7-media                           CentOS-7 - Media                  禁用
centosplus/7/x86_64                CentOS-7 - Plus                   禁用
centosplus-source/7                CentOS-7 - Plus Sources           禁用
cr/7/x86_64                        CentOS-7 - cr                     禁用
extras/7/x86_64                    CentOS-7 - Extras                 启用:   383
extras-source/7                    CentOS-7 - Extras Sources         禁用
fasttrack/7/x86_64                 CentOS-7 - fasttrack              禁用
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
updates/7/x86_64                   CentOS-7 - Updates                启用: 2,053
updates-source/7                   CentOS-7 - Updates Sources        禁用
repolist: 12,069
```

#### 安装

* `yum -y install 包名`，选项：
  * `install`，安装
  * `-y`，自动回答yes 

以安装`yum -y install gcc`编译器为例，如下所示：

```
[root@localhost songyu]# yum -y install gcc
已加载插件：fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/2): extras/7/x86_64/primary_db                          | 168 kB   00:00     
(2/2): updates/7/x86_64/primary_db                         | 6.4 MB   00:02     
Loading mirror speeds from cached hostfile
 * base: ftp.sjtu.edu.cn
 * extras: mirrors.163.com
 * updates: centos.ustc.edu.cn
正在解决依赖关系
--> 正在检查事务
---> 软件包 gcc.x86_64.0.4.8.5-4.el7 将被 升级
--> 正在处理依赖关系 gcc = 4.8.5-4.el7，它被软件包 gcc-gfortran-4.8.5-4.el7.x86_64 需要
--> 正在处理依赖关系 gcc = 4.8.5-4.el7，它被软件包 gcc-c++-4.8.5-4.el7.x86_64 需要
--> 正在处理依赖关系 gcc = 4.8.5-4.el7，它被软件包 libquadmath-devel-4.8.5-4.el7.x86_64 需要
---> 软件包 gcc.x86_64.0.4.8.5-11.el7 将被 更新
--> 正在处理依赖关系 libgomp = 4.8.5-11.el7，它被软件包 gcc-4.8.5-11.el7.x86_64 需要
--> 正在处理依赖关系 cpp = 4.8.5-11.el7，它被软件包 gcc-4.8.5-11.el7.x86_64 需要
--> 正在处理依赖关系 libgcc >= 4.8.5-11.el7，它被软件包 gcc-4.8.5-11.el7.x86_64 需要
--> 正在检查事务
---> 软件包 cpp.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 cpp.x86_64.0.4.8.5-11.el7 将被 更新
---> 软件包 gcc-c++.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 gcc-c++.x86_64.0.4.8.5-11.el7 将被 更新
--> 正在处理依赖关系 libstdc++-devel = 4.8.5-11.el7，它被软件包 gcc-c++-4.8.5-11.el7.x86_64 需要
--> 正在处理依赖关系 libstdc++ = 4.8.5-11.el7，它被软件包 gcc-c++-4.8.5-11.el7.x86_64 需要
---> 软件包 gcc-gfortran.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 gcc-gfortran.x86_64.0.4.8.5-11.el7 将被 更新
--> 正在处理依赖关系 libquadmath = 4.8.5-11.el7，它被软件包 gcc-gfortran-4.8.5-11.el7.x86_64 需要
--> 正在处理依赖关系 libgfortran = 4.8.5-11.el7，它被软件包 gcc-gfortran-4.8.5-11.el7.x86_64 需要
---> 软件包 libgcc.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 libgcc.x86_64.0.4.8.5-11.el7 将被 更新
---> 软件包 libgomp.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 libgomp.x86_64.0.4.8.5-11.el7 将被 更新
---> 软件包 libquadmath-devel.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 libquadmath-devel.x86_64.0.4.8.5-11.el7 将被 更新
--> 正在检查事务
---> 软件包 libgfortran.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 libgfortran.x86_64.0.4.8.5-11.el7 将被 更新
---> 软件包 libquadmath.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 libquadmath.x86_64.0.4.8.5-11.el7 将被 更新
---> 软件包 libstdc++.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 libstdc++.x86_64.0.4.8.5-11.el7 将被 更新
---> 软件包 libstdc++-devel.x86_64.0.4.8.5-4.el7 将被 升级
---> 软件包 libstdc++-devel.x86_64.0.4.8.5-11.el7 将被 更新
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package                  架构          版本                  源           大小
================================================================================
正在更新:
 gcc                      x86_64        4.8.5-11.el7          base         16 M
为依赖而更新:
 cpp                      x86_64        4.8.5-11.el7          base        5.9 M
 gcc-c++                  x86_64        4.8.5-11.el7          base        7.2 M
 gcc-gfortran             x86_64        4.8.5-11.el7          base        6.6 M
 libgcc                   x86_64        4.8.5-11.el7          base         97 k
 libgfortran              x86_64        4.8.5-11.el7          base        295 k
 libgomp                  x86_64        4.8.5-11.el7          base        152 k
 libquadmath              x86_64        4.8.5-11.el7          base        184 k
 libquadmath-devel        x86_64        4.8.5-11.el7          base         47 k
 libstdc++                x86_64        4.8.5-11.el7          base        300 k
 libstdc++-devel          x86_64        4.8.5-11.el7          base        1.5 M

事务概要
================================================================================
升级  1 软件包 (+10 依赖软件包)

总计：38 M
Downloading packages:
警告：/var/cache/yum/x86_64/7/base/packages/gcc-c++-4.8.5-11.el7.x86_64.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY
从 file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 检索密钥
导入 GPG key 0xF4A80EB5:
 用户ID     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 指纹       : 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 软件包     : centos-release-7-2.1511.el7.centos.2.10.x86_64 (@anaconda)
 来自       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在更新    : libgcc-4.8.5-11.el7.x86_64                                 1/22 
  正在更新    : libquadmath-4.8.5-11.el7.x86_64                            2/22 
  正在更新    : libstdc++-4.8.5-11.el7.x86_64                              3/22 
  正在更新    : libstdc++-devel-4.8.5-11.el7.x86_64                        4/22 
  正在更新    : libgfortran-4.8.5-11.el7.x86_64                            5/22 
  正在更新    : libgomp-4.8.5-11.el7.x86_64                                6/22 
  正在更新    : cpp-4.8.5-11.el7.x86_64                                    7/22 
  正在更新    : gcc-4.8.5-11.el7.x86_64                                    8/22 
  正在更新    : libquadmath-devel-4.8.5-11.el7.x86_64                      9/22 
  正在更新    : gcc-gfortran-4.8.5-11.el7.x86_64                          10/22 
  正在更新    : gcc-c++-4.8.5-11.el7.x86_64                               11/22 
  清理        : gcc-gfortran-4.8.5-4.el7.x86_64                           12/22 
  清理        : gcc-c++-4.8.5-4.el7.x86_64                                13/22 
  清理        : libquadmath-devel-4.8.5-4.el7.x86_64                      14/22 
  清理        : gcc-4.8.5-4.el7.x86_64                                    15/22 
  清理        : libgfortran-4.8.5-4.el7.x86_64                            16/22 
  清理        : libstdc++-devel-4.8.5-4.el7.x86_64                        17/22 
  清理        : libstdc++-4.8.5-4.el7.x86_64                              18/22 
  清理        : libgcc-4.8.5-4.el7.x86_64                                 19/22 
  清理        : libquadmath-4.8.5-4.el7.x86_64                            20/22 
  清理        : cpp-4.8.5-4.el7.x86_64                                    21/22 
  清理        : libgomp-4.8.5-4.el7.x86_64                                22/22 
  验证中      : gcc-c++-4.8.5-11.el7.x86_64                                1/22 
  验证中      : libquadmath-4.8.5-11.el7.x86_64                            2/22 
  验证中      : cpp-4.8.5-11.el7.x86_64                                    3/22 
  验证中      : libgfortran-4.8.5-11.el7.x86_64                            4/22 
  验证中      : libquadmath-devel-4.8.5-11.el7.x86_64                      5/22 
  验证中      : libgcc-4.8.5-11.el7.x86_64                                 6/22 
  验证中      : libstdc++-4.8.5-11.el7.x86_64                              7/22 
  验证中      : gcc-gfortran-4.8.5-11.el7.x86_64                           8/22 
  验证中      : libgomp-4.8.5-11.el7.x86_64                                9/22 
  验证中      : gcc-4.8.5-11.el7.x86_64                                   10/22 
  验证中      : libstdc++-devel-4.8.5-11.el7.x86_64                       11/22 
  验证中      : gcc-4.8.5-4.el7.x86_64                                    12/22 
  验证中      : gcc-c++-4.8.5-4.el7.x86_64                                13/22 
  验证中      : libstdc++-4.8.5-4.el7.x86_64                              14/22 
  验证中      : libquadmath-devel-4.8.5-4.el7.x86_64                      15/22 
  验证中      : libgcc-4.8.5-4.el7.x86_64                                 16/22 
  验证中      : libgfortran-4.8.5-4.el7.x86_64                            17/22 
  验证中      : libstdc++-devel-4.8.5-4.el7.x86_64                        18/22 
  验证中      : gcc-gfortran-4.8.5-4.el7.x86_64                           19/22 
  验证中      : cpp-4.8.5-4.el7.x86_64                                    20/22 
  验证中      : libgomp-4.8.5-4.el7.x86_64                                21/22 
  验证中      : libquadmath-4.8.5-4.el7.x86_64                            22/22 

更新完毕:
  gcc.x86_64 0:4.8.5-11.el7                                                     

作为依赖被升级:
  cpp.x86_64 0:4.8.5-11.el7            gcc-c++.x86_64 0:4.8.5-11.el7            
  gcc-gfortran.x86_64 0:4.8.5-11.el7   libgcc.x86_64 0:4.8.5-11.el7             
  libgfortran.x86_64 0:4.8.5-11.el7    libgomp.x86_64 0:4.8.5-11.el7            
  libquadmath.x86_64 0:4.8.5-11.el7    libquadmath-devel.x86_64 0:4.8.5-11.el7  
  libstdc++.x86_64 0:4.8.5-11.el7      libstdc++-devel.x86_64 0:4.8.5-11.el7    

完毕！
```

#### 升级

* `yum -y update 包名`，升级：
  * `update`，升级
  * `-y`，自动回答yes

#### 卸载

* `yum -y remove 包名`，卸载需谨慎：
  * `remove`，卸载
  * `-y`，自动回答yes

### yum软件组管理命令

* `yum grouplist`，列出所有可用的软件组列表
* `yum groupinstall 软件组名`，安装指定软件组，组名可通过字符界面下运行`yum grouplist`命令查询，组名不能是中文
* `yum groupremove 软件组名`，卸载指定软件组

```
[root@localhost songyu]# yum grouplist
已加载插件：fastestmirror, langpacks
没有安装组信息文件
Maybe run: yum groups mark convert (see man yum)
Loading mirror speeds from cached hostfile
 * base: ftp.sjtu.edu.cn
 * extras: mirrors.163.com
 * updates: centos.ustc.edu.cn
可用的环境分组：
   最小安装
   基础设施服务器
   计算节点
   文件及打印服务器
   基本网页服务器
   虚拟化主机
   带 GUI 的服务器
   GNOME 桌面
   KDE Plasma Workspaces
   开发及生成工作站
可用组：
   传统 UNIX 兼容性
   兼容性程序库
   图形管理工具
   安全性工具
   开发工具
   控制台互联网工具
   智能卡支持
   科学记数法支持
   系统管理
   系统管理工具
完成
```

## 源码包

源码包与rpm包的区别：
* 安装之前的区别，概念上不同
  * rpm包不开源
  * 源码包开源
* 安装之后的区别，安装位置不同
  * rpm包不建议指定安装位置，而是使用默认的安装位置，安装的服务可以使用系统服务管理命令(service)来管理，例如rpm包安装的apache的启动方法是
```
/etc/rc.d/init.d/httpd start
```
```
service httpd start
```

  * 因为源码包没有卸载命令，所以源码包必须安装在指定位置当中，一般选择安装在
```
/usr/local/软件名/
```
并且源码包安装的服务不能被服务管理命令管理，因为没有安装到默认路径中，只能用绝对路径进行服务的管理，如
```
/usr/local/apache2/bin/apachectl start
```

源码包的特点是开源、自定义、本机编译的效率高，而rpm包由厂商编译，在本机执行时的效率不一定高。

### 安装

* 安装前的准备工作：
  * 安装C语言编译器，因为Linux中源码包都是以C语言编写，所以在安装源码包之前，必须预先安装C语言编译器，推荐使用yum方式安装
```
yum -y install gcc
```
```
[root@localhost songyu]# rpm -qa | grep gcc
gcc-c++-4.8.5-11.el7.x86_64
gcc-4.8.5-11.el7.x86_64
libgcc-4.8.5-11.el7.x86_64
gcc-gfortran-4.8.5-11.el7.x86_64
```
  * 下载源码包
* 安装时的注意事项：
  * 源代码保存位置，`/usr/local/src/`
  * 软件安装位置，`/usr/local/`
  * 如何确定安装过程报错？安装过程停止，并出现error、warning或no的提示  
* 安装过程：
  1. 解压缩下载的源码包
  2. 进入解压缩目录
  3. `./configure`，软件配置与检查
    * 定义需要的功能选项，`./configure --help`查看选项，源码包必须指定安装位置
```
./configure --prefix=/usr/local/apache2
```
    * 检测系统环境是否符合安装要求
    * 把定义好的功能选项和检测系统环境的信息都写入`Makefile`文件，用于后续的编辑 
  4. `make`，编译
    * 编译报错时使用`make clean`命令，清除编译产生的缓存文件
  5. `make install`，安装
  6. 启动

### 卸载

源码包的卸载不需要卸载命令，直接删除安装目录即可，不会遗留任何垃圾文件。

## 一键安装包

所谓的一键安装包，实际上还是安装的rpm包与源码包，只是把安装过程写成了脚本，便于安装
* 优点：简单、快速、方便
* 缺点：不能定义安装软件的版本、不能定义所需要的软件功能、源码包的优势丧失

* [参考：慕课网](http://www.imooc.com/course/list?c=linux)