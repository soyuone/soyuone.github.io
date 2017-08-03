---
layout: post
title: "apache在windows平台的配置及常用命令"
categories: 服务器
tags: 服务器 apache windows 负载均衡
author: Kopite
---

* content
{:toc}


apache`版本号apache/2.4.27`在windows平台的配置及常用命令。



## 下载和安装

* 进入[Apache HTTP Server官方网站](http://httpd.apache.org/)，选择合适的Apache HTTP Server版本进行下载，此处选择下载`2.4.27版本`，单击`Download`
* 单击`Files for Microsoft Windows`
* Apache HTTP Server项目只提供源码，并不提供编译后的安装包，因此从`Downloading Apache for Windows`中列出的对Apache HTTP Server提供编译后安装包的第三方网站中选择一个进行下载，此处选择下载`ApacheHaus`，单击该连接[ApacheHaus](https://www.apachehaus.com/cgi-bin/download.plx)
* 选择所需要的32位或者64位版本进行下载，此处选择下载64位版本，下载后得到`httpd-2.4.27-x64-vc14.zip`文件
* 将该压缩包解压至`D:\httpd-2.4.27-x64-vc14`
* 命令行下进入`D:\httpd-2.4.27-x64-vc14\Apache24\bin`目录

* [参考：如何从apache官网下载windows版apache服务器](https://jingyan.baidu.com/article/29697b912f6539ab20de3cf8.html)
* [参考：Apache的httpd命令详解](http://www.cnblogs.com/azhw/p/6170949.html)