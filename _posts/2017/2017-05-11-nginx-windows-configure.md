---
layout: post
title: "nginx在windows平台的配置及常用命令"
categories: 服务器
tags: 服务器 nginx windows 负载均衡
author: Kopite
---

* content
{:toc}


nginx`版本号nginx/1.11.11`在windows平台的配置及常用命令。



## 配置nginx

下载nginx windows版本后解压，运行命令行并切换至解压路径，输入`D:\nginx-1.11.11>start nginx`，可以看到一个窗口一闪而过，此时nginx即被开启，可以在任务管理器中找到它的进程，在浏览器中输入`localhost`或者`localhost:80`能看到欢迎页面。

## 常用命令
```
D:\nginx-1.11.11>start nginx //启动nginx
```
```
D:\nginx-1.11.11>nginx -v //查看nginx版本号
nginx version: nginx/1.11.11
```
```
D:\nginx-1.11.11>nginx.exe -s stop //快速停止nginx，可能并不保存相关信息
```
```
D:\nginx-1.11.11>nginx.exe -s quit //正常停止nginx，保存相关信息
```
```
D:\nginx-1.11.11>nginx.exe -s reload //配置信息修改，需要重新载入配置信息时使用此命令
```
```
D:\nginx-1.11.11>tasklist /fi "imagename eq nginx.exe" //查看windows任务管理器中nginx的进程

映像名称                       PID 会话名              会话#       内存使用
========================= ======== ================ =========== ============
nginx.exe                    13996 Console                   34      6,536 K
nginx.exe                    13900 Console                   34      7,164 K
```