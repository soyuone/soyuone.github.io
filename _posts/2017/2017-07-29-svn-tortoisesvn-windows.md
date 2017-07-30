---
layout: post
title: "使用SVN进行协作开发"
categories: SVN
tags: SVN windows
author: Kopite
---

* content
{:toc}


对于普通开发者而言，通常会选择使用`TortoiseSVN`作为SVN的客户端，本文对TortoiseSVN`版本号1.9.6`在windows平台的使用进行整理。
<br>
<br>
TortoiseSVN与普通软件不同，它并未提供任何窗口来执行版本管理操作，只是一个`Shell`扩展，并且已经被整合到windows资源管理器中，使用TortoiseSVN时，只需要右键单击任何文件夹、文件，即可弹出TortoiseSVN工具菜单。



## 下载和安装

* 登录[TortoiseSVN官方网站](https://tortoisesvn.net/)，选择合适的TortoiseSVN版本进行下载，此处选择下载`1.9.6版本`，下载后得到`TortoiseSVN-1.9.6.27867-x64-svn-1.9.6.msi`文件
* `Next` -> `Next`
* 选择安装路径
* 单击`Install`，开始安装
* 单击`Finish`，安装完成

## 建立SVN资源库

* 新建文件夹，例如`E:\svnRepository`
* 打开文件夹，在空白处右键选择`TortoiseSVN` -> `Create repository here`
* 弹出对话框，提示创建成功，单击`OK`，自动创建目录结构，如下图所示

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-1.png)

## 将项目发布到服务器

使用TortoiseSVN将项目发布到服务器，假设在磁盘上开发了一个`svnProject`项目，现在希望将该项目发布到SVN服务器上，按如下步骤进行：
* 右键单击该项目对应的文件夹，即`D:\workspace\svnProject`
* 在弹出的菜单中，选择`TortoiseSVN` -> `Import`
* 在`URL of repository:`处输入SVN资源库的URL，在`Import message`处填写备注信息，如下图所示

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-2.png)

* 单击`OK`开始发布
* 发布完成后弹出`Import Finished!`提示，可以查看发布的文件，单击`OK`，将项目发布到SVN服务器完成，如下图所示

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-3.png)

## 从服务器下载项目

从服务器下载项目，如果项目组其他开发者想从服务器上下载刚刚发布的`svnProject`项目，按如下步骤进行：
* 进入用于保存待下载项目的目标磁盘空间，例如`D:\workspace\svnDownload`
* 在空白处右键单击，选择`SVN Checkout...`
* 在弹出的`Checkout`对话框`URL of repository:`处输入SVN资源库的URL，在`Checkout directory:`处指定将项目下载到本地磁盘的指定位置，点击`OK`开始下载，如下图所示

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-4.png)

* 项目下载成功后弹出`Checkout Finished!`提示，单击`OK`，如下图所示

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-5.png)

从SVN服务器上下载的项目、文件上都有一个绿色的小钩，这表明该项目、该文件与服务器项目状态一致。

## 提交修改

从服务器下载了指定项目之后，开发者就可以对项目进行开发，当项目开发到一定阶段后，接下来应该将自己手中所做的修改提交到服务器，这样其他开发者也能得到该项目最新修改过的版本。
<br>
<br>
SVN项目修改过的文件的图标以及该文件所在文件夹的图标都会增加一个红色的感叹号，这表明该文件或该文件夹被修改过，需要将该修改提交到服务器，提交修改按如下步骤进行：
* 
* 右键单击项目对应的文件夹

[参考：TortoiseSVN的下载、安装以及常用操作](https://jingyan.baidu.com/article/358570f6638aa4ce4724fcf7.html)