---
layout: post
title: "Linux中的软件安装"
categories: Linux
tags: Linux
author: Kopite
---

* content
{:toc}


Windows中的软件在Linux中无法安装，并且生产环境下的Linux服务器通常没有图形用户界面，需要在字符界面下安装软件。



## 概述

Linux中的软件包分为`源码包`和`二进制包（rpm包、系统默认包）`。
<br>
<br>
所谓的`脚本安装包`，就是把复杂的软件包安装过程写成了程序脚本，初学者可以执行程序脚本实现一键安装，但实际安装的还是源码包和二进制包：
* 优点：安装简单，快捷
* 缺点：完全丧失了自定义性

## rpm命令管理

### rpm包命名规则

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

* [参考：慕课网](http://www.imooc.com/course/list?c=linux)