---
layout: post
title: "tomcat在windows平台的常用配置"
categories: 服务器
tags: 服务器 tomcat windows
author: Kopite
---

* content
{:toc}


tomcat`版本号Apache Tomcat/7.0.65`在windows平台的常用配置。



## 配置环境变量

* 新建变量`CATALINA_HOME`，添加变量值`D:\apache-tomcat-7.0.65`
* 新建变量`CATALINA_BASE`，添加变量值`D:\apache-tomcat-7.0.65`
* 在变量`Path`的变量值中添加`%CATALINA_HOME%\lib;%CATALINA_HOME%\bin;`
* 测试，运行命令行并输入`C:\Users\SOYU>startup`或者切换至`\bin>`目录下输入`E:\cluster\apache-tomcat-7.0.65\bin>startup`，在浏览器的地址栏中输入`http://localhost:8080`，此时运行的是`\webapps\ROOT`路径中的项目

## 配置一机多tomcat

将tomcat解压至不同路径下，分别配置环境变量，此处以配置两个tomcat为例。tomcat 1的配置与配置一个tomcat时相同，tomcat 2的配置如下
* 新建变量`CATALINA_HOME_2`，添加变量值`E:\cluster\apache-tomcat-7.0.65`
* 新建变量`CATALINA_BASE_2`，添加变量值`E:\cluster\apache-tomcat-7.0.65`
* 此处不再在变量`Path`的变量值中添加`%CATALINA_HOME_2%\lib;%CATALINA_HOME_2%\bin;`，运行命令行时需要切换至`\bin`目录下输入
* 修改`\bin`目录下`catalina.bat`、`startup.bat`、`shutdown.bat`三个文件中所有的`CATALINA_BASE`、`CATALINA_HOME`，将其修改至与环境变量中的名称相同
* 修改`\conf\server.xml`文件中以下三处的port

```xml
<Server port="8007" shutdown="SHUTDOWN">


<Connector URIEncoding="UTF-8" connectionTimeout="20000" disableUploadTimeout="true" port="8082" protocol="HTTP/1.1" redirectPort="8443"/>


<Connector port="8010" protocol="AJP/1.3" redirectPort="8443"/>
```

启动、停止时，在命令行中分别切换至各tomcat解压路径`\bin`目录下，运行`startup.bat`、`shutdown.bat`命令

## 修改默认首页

将`\webapps\ROOT`路径下的文件全部删除后，放入项目代码，输入`http://localhost:8080`出现的是新置入项目的默认首页

## 配置用户

tomcat在访问`http://localhost:8080/`下图所示的三个功能时需要配置用户，否则会报错
![](/image/2017/2017-05-11-tomcat-windows-configure-1.png)
tomcat 7中配置用户，在`\conf\tomcat-users.xml`中添加以下配置

```
<tomcat-users>
<!--
  NOTE:  By default, no user is included in the "manager-gui" role required
  to operate the "/manager/html" web application.  If you wish to use this app,
  you must define such a user - the username and password are arbitrary.
-->

	<role rolename="manager"/>  
	<role rolename="manager-gui"/>  
	<role rolename="admin"/>  
	<role rolename="admin-gui"/> 
    <user username="tomcat" password="admin" roles="admin-gui,admin,manager-gui,manager"/>
</tomcat-users>
```

## 虚拟目录

可以通过多个路径访问tomcat中的同一个项目，在`‪D:\apache-tomcat-7.0.65\conf\server.xml`中添加如下配置

```xml
<Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true">

<!-- SingleSignOn valve, share authentication between web applications
     Documentation at: /docs/config/valve.html -->
<!--
<Valve className="org.apache.catalina.authenticator.SingleSignOn" />
-->

<!-- Access log processes all example.
     Documentation at: /docs/config/valve.html
     Note: The pattern used is equivalent to using pattern="common" -->
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" pattern="%h %l %u %t &quot;%r&quot; %s %b" prefix="localhost_access_log." suffix=".txt"/>

<Context path="/videoplatform" docBase="videoplatform" reloadable="true"/>
<Context path="/platform" docBase="videoplatform" reloadable="true"/>
</Host>
```

即可通过如下两种方式访问该项目
* `http://localhost:8080/platform/`
* `http://localhost:8080/videoplatform/`

`Host`中相关属性
* `appBase`
* `autoDeploy`
* `name`
* `unpackWARs`

`Context`中相关属性
* `path`
* `docBase`
* `reloadable`‪