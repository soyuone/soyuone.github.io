---
layout: post
title: "nginx+tomcat+spring session+redis在windows平台实现负载均衡及session共享"
categories: 服务器
tags: 服务器 nginx tomcat NoSQL redis windows session 负载均衡
author: Kopite
---

* content
{:toc}


nginx`版本号nginx/1.11.11`，jedis`版本号2.9.0`，spring-data-redis`版本号1.7.1.RELEASE`，spring session`版本号1.3.0.RELEASE`，如下所示
```xml
<!-- nosql -->
<jedis.version>2.9.0</jedis.version>
<spring-data-redis.version>1.7.1.RELEASE</spring-data-redis.version>
<spring-session.version>1.3.0.RELEASE</spring-session.version>

<!-- redis begin -->
<dependency>
	<groupId>redis.clients</groupId>
	<artifactId>jedis</artifactId>
	<version>${jedis.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-redis</artifactId>
	<version>${spring-data-redis.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework.session</groupId>
	<artifactId>spring-session</artifactId>
	<version>${spring-session.version}</version>
</dependency>
```



## nginx简介

nginx是一款轻量级兼备高性能的http服务器、反向代理服务器
* `反向代理`是指客户端本可以直接通过http协议访问某网站应用服务器，如果网站管理员在中间加上一个nginx，客户端请求nginx，nginx请求应用服务器，然后将结果返回给客户端，此时nginx就是反向代理服务器
* `负载均衡`是指反向代理服务器将接收到的请求分发到部署的多台服务器中，将大量用户的请求分配给多台机器处理，假设在运行过程中其中一台服务器挂掉，还有其它服务器可以访问，更新的时候也可以先只关闭其中一台，轮流进行更新

## 配置spring session、redis

* 配置spring session

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://www.springframework.org/schema/beans" xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="
	http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
    http://www.springframework.org/schema/util
    http://www.springframework.org/schema/util/spring-util-4.2.xsd">

	<!-- 使用spring-session把HttpSession放入Redis中，从而实现HttpSession共享，http://www.cnblogs.com/avivaye/p/4935137.html 
		http://blog.csdn.net/wwd0501/article/details/51484671 http://blog.csdn.net/zdsdiablo/article/details/50428212 
		http://blog.csdn.net/xiejx618/article/details/42919327 -->
	<bean
		class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration">
		<!-- Redis-Key的有效期，不是 HttpSession的有效期，默认值：1800秒 -->
		<property name="maxInactiveIntervalInSeconds" value="172800" />
	</bean>

	<!-- 让spring-session不再执行config命令，http://blog.csdn.net/xiao__gui/article/details/52706243 -->
<!-- 	<util:constant
		static-field="org.springframework.session.data.redis.config.ConfigureRedisAction.NO_OP" /> -->
</beans>
```

* 配置redis

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="
	http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context-4.2.xsd  
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx-4.2.xsd">

	<!-- jedis连接池，http://www.cnblogs.com/tankaixiong/p/4048167.html -->
	<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<!-- 最大空闲连接数量，默认值：8 -->
		<property name="maxIdle" value="${redis.maxIdle}" />
		<!-- 最小空闲连接数量，默认值：0 -->
		<property name="minIdle" value="${redis.minIdle}" />
		<!-- jedis pool中的最大连接数量，默认值：8 -->
		<property name="maxTotal" value="${redis.maxTotal}" />
		<!-- 获取连接时的最大等待时长（毫秒），默认值：-1L -->
		<property name="maxWaitMillis" value="${redis.maxWaitMillis}" />
		<!-- jedis pool中可用资源耗尽时是否阻塞，false：报异常，true：阻塞直到超时，默认true -->
		<property name="blockWhenExhausted" value="${redis.blockWhenExhausted}" />
		<!-- 获取连接时检查有效性，默认值：false -->
		<property name="testOnBorrow" value="${redis.testOnBorrow}" />
		<!-- 是否检测空闲连接的有效性，默认值：false -->
		<property name="testWhileIdle" value="${redis.testWhileIdle}" />
		<!-- 空闲对象被清除需要达到的最小空闲时间，默认值：1800000毫秒 -->
		<property name="minEvictableIdleTimeMillis" value="${redis.minEvictableIdleTimeMillis}" />
		<!-- 逐出扫描的时间间隔（毫秒），如果为负数则不运行逐出线程，默认值：-1 -->
		<property name="timeBetweenEvictionRunsMillis" value="${redis.timeBetweenEvictionRunsMillis}" />
	</bean>

	<!-- jedis连接工厂 -->
	<bean id="jedisConnectionFactory"
		class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<property name="hostName" value="${redis.hostName}" />
		<property name="port" value="${redis.port}" />
		<property name="password" value="${redis.password}" />
		<property name="database" value="${redis.database}" />
		<property name="timeout" value="${redis.timeout}" />
		<property name="poolConfig" ref="jedisPoolConfig" />
	</bean>

	<!-- Spring对Redis的封装，http://www.cnblogs.com/tankaixiong/p/3660075.html http://www.tuicool.com/articles/3aAbMz -->
	<!-- spring-data-redis版本问题，http://blog.csdn.net/forlovedoit/article/details/52692910 -->
	<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
		<property name="connectionFactory" ref="jedisConnectionFactory" />
	</bean>
</beans>
```

* 登录成功后，将session放入redis

```java
// 使用spring-session把HttpSession放入Redis中
HttpSession httpSession = request.getSession();
httpSession.setAttribute("_USER_", userVO);
```

## 部署应用到tomcat

为实现负载均衡，此处在同一台电脑上安装两个tomcat，分别位于`D:\apache-tomcat-7.0.65`、`E:\cluster\apache-tomcat-7.0.65`，端口分别设为`8080`、`8082`。将应用分别部署到tomcat`\webapps`路径下，为区分这两个tomcat，对`usercenter.jsp`页面加以修改。

* tomcat 1中的`usercenter.jsp`页面如下所示

```
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Hospital</title>
</head>
<body>
	<h1 style="text-align: center">这是${_USER_.email}的个人中心页面，位于Tomcat 1</h1>
	<div style="text-align: center">
		<p style="font-size: 15px">欢迎您，尊敬的${_USER_.username}</p>
	</div>
</body>
</html>
```

* tomcat 2中的`usercenter.jsp`页面如下所示

```
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Hospital</title>
</head>
<body>
	<h1 style="text-align: center">这是${_USER_.email}的个人中心页面，位于Tomcat 2</h1>
	<div style="text-align: center">
		<p style="font-size: 15px">欢迎您，尊敬的${_USER_.username}</p>
	</div>
</body>
</html>
```

## 配置nginx

* 修改`\conf\nginx.conf`文件
```
	upstream mynginx{
		server 127.0.0.1:8080;
		server 127.0.0.1:8082;
		
	}
	
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
			proxy_pass  http://mynginx;
            #root   html;
            #index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        #error_page   500 502 503 504  /50x.html;
        #location = /50x.html {
        #    root   html;
        #}

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
```

重启nginx，在浏览器输入`http://localhost/hospital/login.jsp`，登录成功后多刷新几次，可以看到轮流展示两个`usercenter.jsp`页面。

* [本文所用的完整项目代码托管在github](https://github.com/soyuone/hospital)
* [参考：nginx入门](http://blog.csdn.net/u012486840/article/details/53098890)
* [参考：tomcat+nginx+redis实现均衡负载、session共享(一)](http://www.cnblogs.com/zhrxidian/p/5432886.html)
* [参考：tomcat+nginx+redis实现均衡负载、session共享(二)](http://www.cnblogs.com/zhrxidian/p/5491285.html#3619379)



