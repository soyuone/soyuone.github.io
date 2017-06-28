---
layout: post
title: "jdk在windows平台的安装和配置"
categories: java
tags: java windows
author: Kopite
---

* content
{:toc}


jdk-7u79-windows-x64`版本号1.7.0_79`在windows平台的安装和配置。



## 安装jdk

![](/image/2017/2017-05-10-java-setup-jdk-1.png)

* 不需要安装公共JRE
* 上图为jdk-8u25-windows-x64的安装截图

## 配置环境变量

* 新建变量`JAVA_HOME`，添加变量值`D:\Java\jdk1.7.0_79`
* 新建变量`CLASSPATH`，添加变量值`.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;`（提供程序在运行期间寻找所需资源的路径）
* 在变量`Path`的变量值中添加`%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;`（提供操作系统寻找java命令工具的路径）



