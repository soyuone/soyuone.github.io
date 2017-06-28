---
layout: post
title: "maven在windows平台的安装和配置"
categories: maven
tags: maven windows
author: Kopite
---

* content
{:toc}


maven`版本号3.3.3`在windows平台的安装和配置。



## 常用配置

* 新建变量`M2_HOME`，添加变量值`D:\maven\apache-maven-3.3.3`
* 在变量`Path`的变量值中添加`%M2_HOME%\bin;`
* 测试，打开命令行输入`mvn -v`
* 打开\conf\settings.xml文件，修改本地仓库的位置、修改maven中央仓库的国内镜像、修改maven的jdk版本

```xml
<localRepository>D:\mavenRepository</localRepository>
```

```xml
  <mirrors>
	<mirror>  
		<id>alimaven</id>  
		<name>aliyun maven</name>  
		<url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
		<mirrorOf>central</mirrorOf>          
	</mirror>
  </mirrors>
```

```
<profile>
  <id>jdk-1.7</id>

  <activation>
	<activeByDefault>true</activeByDefault>
	<jdk>1.7</jdk>
  </activation>

  <properties>
	<maven.compiler.source>1.7</maven.compiler.source>
	<maven.compiler.target>1.7</maven.compiler.target>
	<maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>
  </properties>
</profile>
```
