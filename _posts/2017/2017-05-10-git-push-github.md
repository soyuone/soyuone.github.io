---
layout: post
title: "使用git将项目上传至github"
categories: 版本控制
tags: 版本控制 git github
author: Kopite
---

* content
{:toc}


使用git`版本号1.9.5.msysgit.1`将项目上传至github。



## 上传过程

* 在github中选择`New repository`，创建完毕后可以得到一个URL
* 在项目根目录下鼠标右键，选择`Git Init Here`或者选择`Git Bash`后输入`$ git init`
* 鼠标右键，选择`Git Gui`，在远端(remote)中选择Add，如下图所示，选择Add<br>
![](/image/2017/2017-05-10-git-push-github-1.png)
* 鼠标右键，选择`Git Add all files now`或者在`Git Gui`界面中选择`缓存改动`，添加描述后点击`提交`
* 选择`上传`，选中`强制覆盖已有的分支`，选择上传，根据提示输入用户名及密码后即上传成功
* [参考：将本地文件上传至github](http://jingyan.baidu.com/article/27fa732683ebf546f8271f2e.html)