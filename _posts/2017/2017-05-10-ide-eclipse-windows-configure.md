---
layout: post
title: "eclipse在windows平台的常用配置"
categories: 开发工具
tags: 开发工具 eclipse windows
author: Kopite
---

* content
{:toc}


eclipse`版本号Luna Service Release 2 (4.4.2)`在windows平台的常用配置。



## 配置jdk

![](/image/2017/2017-05-10-ide-eclipse-windows-configure-1.png)

![](/image/2017/2017-05-10-ide-eclipse-windows-configure-2.png)

## 配置tomcat

![](/image/2017/2017-05-10-ide-eclipse-windows-configure-3.png)

为解决get请求时传中文参数乱码问题，在\conf\server.xml文件Connector标签中添加`URIEncoding="UTF-8"`参数。

```xml
<Connector URIEncoding="UTF-8" connectionTimeout="20000" disableUploadTimeout="true" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```

## 配置maven

![](/image/2017/2017-05-10-ide-eclipse-windows-configure-4.png)

![](/image/2017/2017-05-10-ide-eclipse-windows-configure-5.png)

![](/image/2017/2017-05-10-ide-eclipse-windows-configure-2.png)

在Default VM arguments中添加 `-Dmaven.multiModuleProjectDirectory=$M2_HOME`