---
layout: post
title: "druid数据库连接池"
categories: 数据源
tags: 数据源 druid
author: Kopite
---

* content
{:toc}


druid，为监控而生的数据库连接池，能够提供强大的监控和扩展功能。由于druid有对sql执行的监控功能，项目中使用的连接池由c3p0更换为druid，[各种连接池性能对比测试](https://github.com/alibaba/druid/wiki/%E5%90%84%E7%A7%8D%E8%BF%9E%E6%8E%A5%E6%B1%A0%E6%80%A7%E8%83%BD%E5%AF%B9%E6%AF%94%E6%B5%8B%E8%AF%95)。



## 配置依赖

```xml
<!-- http://www.mvnrepository.com/artifact/com.alibaba/druid -->
<druid.version>1.0.18</druid.version>

<!-- druid -->
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid</artifactId>
	<version>${druid.version}</version>
</dependency>
```

## 配置StatViewServlet

[druid内置提供了一个`StatViewServlet`用于展示druid的统计信息](https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE)，用途包括
* 提供监控信息展示的html页面
* 提供监控信息的JSON API

`StatViewServlet`是一个标准的`javax.servlet.http.HttpServlet`，需要配置在web应用的`WEB-INF/web.xml`文件中，如下所示。

```xml
<!-- StatViewServlet配置，https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE -->
<servlet>
	<servlet-name>DruidStatView</servlet-name>
	<servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
	<init-param>
		<!-- 允许清空统计数据 -->
		<param-name>resetEnable</param-name>
		<param-value>true</param-value>
	</init-param>
	<init-param>
		<!-- 用户名 -->
		<param-name>loginUsername</param-name>
		<param-value>druid</param-value>
	</init-param>
	<init-param>
		<!-- 密码 -->
		<param-name>loginPassword</param-name>
		<param-value>druid</param-value>
	</init-param>
</servlet>

<servlet-mapping>
	<servlet-name>DruidStatView</servlet-name>
	<url-pattern>/druid/*</url-pattern>
</servlet-mapping>
```

* 通过`resetEnable`参数，可以在监控页面中点击`重置`，将所有计数器清零、重新计数，赋值为`false`时关闭重置功能
* 通过`loginUsername`、`loginPassword`这两个初始参数，可以配置监控页面的登录访问密码
* 通过`allow`、`deny`这两个参数，可以进行ip访问控制。`allow`参数没有配置或者为空时，允许所有ip访问。`deny`参数优先于`allow`参数，如果ip在`deny`列表中，就算在`allow`列表，访问也会被拒绝
* 根据配置的`url-pattern`来访问内置监控页面：`http://<host>:<port>/<context>/druid/`，本示例中：
```
http://localhost:8080/hospital/druid/
```

监控登录页面：<br>
![](/image/2017/2017-05-12-druid-how-to-use-1.png)

## 配置WebStatFilter

[`WebStatFilter`用于采集web-jdbc关联监控的数据](https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_%E9%85%8D%E7%BD%AEWebStatFilter)，需要配置在web应用的`WEB-INF/web.xml`文件中，如下所示。

```xml
<!-- Druid数据库连接池，https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98 
	WebStatFilter配置，https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_%E9%85%8D%E7%BD%AEWebStatFilter -->
<filter>
	<filter-name>DruidWebStatFilter</filter-name>
	<filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
	<init-param>
		<param-name>exclusions</param-name>
		<param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
	</init-param>
	<init-param>
		<param-name>sessionStatMaxCount</param-name>
		<param-value>1000</param-value>
	</init-param>
	<init-param>
		<param-name>sessionStatEnable</param-name>
		<param-value>false</param-value>
	</init-param>
</filter>

<filter-mapping>
	<filter-name>DruidWebStatFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

* 通过`exclusions`参数，可以排除一些不必要的url，比如*.js,/jslib/*等
* `sessionStatMaxCount`参数缺省时，默认值：`1000`
* `sessionStatEnable`参数，是否开启session统计功能
* `principalSessionName`、`principalCookieName`、`profileEnable`等参数的使用详见wiki

## 配置dataSource

druid.properties
```properties
#druid
druid.url=jdbc:mysql://localhost:3306/hospital?useUnicode=true&characterEncoding=UTF-8
druid.username=root
druid.password=XY+AwyimCIuvPl5Njm7buayg8zp7TwIGX7RAufaR9KQnCSpR6fVp2vvA7KQOJYP/VOVPbmT8wyjmvi6GlUDhvA==
druid.decrypt=true
druid.publickey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKgi+EHD6RHDPuzong8eohKCSe66ZwXpr+vYKhqlnYrY0VyMQR9aT2anjdv75GRGEWWMqW4YgpxiCImUnl6vHXECAwEAAQ==

druid.initialSize=1
druid.maxActive=50
druid.minIdle=1
druid.maxWait=10000
druid.timeBetweenEvictionRunsMillis=30000
druid.minEvictableIdleTimeMillis=60000

druid.validationQuery=select 1
druid.testOnBorrow=true
druid.testOnReturn=false
druid.testWhileIdle=true

druid.removeAbandoned=true
druid.removeAbandonedTimeout=1800
druid.logAbandoned=true

druid.poolPreparedStatements=false
druid.maxPoolPreparedStatementPerConnectionSize=20

#if password is encrypted, config is need.
druid.filters=config,wall
```

spring-datasource-druid.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:task="http://www.springframework.org/schema/task"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="
	http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context-4.2.xsd  
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
    http://www.springframework.org/schema/task  
    http://www.springframework.org/schema/task/spring-task-4.2.xsd 
    http://www.springframework.org/schema/util
    http://www.springframework.org/schema/util/spring-util-4.2.xsd">

	<!-- Druid数据库连接池，http://www.open-open.com/lib/view/open1430558786084.html 
		http://www.cnblogs.com/wuyun-blog/p/5679073.html -->
	<bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
		init-method="init" destroy-method="close">
		<!-- 基本属性 url、username、password，https://www.oschina.net/question/873438_234580 -->
		<property name="url" value="${druid.url}" />
		<property name="username" value="${druid.username}" />
		<property name="password" value="${druid.password}" />
		<!-- 提示Druid数据源需要对数据库密码进行解密，https://github.com/alibaba/druid/wiki/%E4%BD%BF%E7%94%A8ConfigFilter -->
		<property name="connectionProperties"
			value="config.decrypt=${druid.decrypt};config.decrypt.key=${druid.publickey}" />

		<!-- 初始化连接池连接值、连接池中连接最大值、连接池中连接最小值 -->
		<property name="initialSize" value="${druid.initialSize}" />
		<property name="maxActive" value="${druid.maxActive}" />
		<property name="minIdle" value="${druid.minIdle}" />
		<!-- 获取连接时等待最大等待时间（毫秒） -->
		<property name="maxWait" value="${druid.maxWait}" />
		<!-- 1、检测需要关闭的空闲连接的间隔时间（毫秒）.2、testWhileIdle的判断依据. 默认值：60 * 1000L -->
		<property name="timeBetweenEvictionRunsMillis" value="${druid.timeBetweenEvictionRunsMillis}" />
		<!-- 一个连接在池中的最小生存时间（毫秒），默认值：1000L * 60L * 30L -->
		<property name="minEvictableIdleTimeMillis" value="${druid.minEvictableIdleTimeMillis}" />

		<!-- 连接有效性测试 -->
		<property name="validationQuery" value="${druid.validationQuery}" />
		<property name="testOnBorrow" value="${druid.testOnBorrow}" />
		<property name="testOnReturn" value="${druid.testOnReturn}" />
		<property name="testWhileIdle" value="${druid.testWhileIdle}" />

		<!-- 是否强制关闭长时间不使用的连接 -->
		<property name="removeAbandoned" value="${druid.removeAbandoned}" />
		<!-- 超过*秒关闭空闲连接 -->
		<property name="removeAbandonedTimeout" value="${druid.removeAbandonedTimeout}" />
		<!-- 将当前关闭动作记录到日志 -->
		<property name="logAbandoned" value="${druid.logAbandoned}" />

		<!-- 是否缓存preparedStatement（PSCache），默认值：false；指定每个连接上的PSCache的大小，默认值：10 -->
		<property name="poolPreparedStatements" value="${druid.poolPreparedStatements}" />
		<property name="maxPoolPreparedStatementPerConnectionSize"
			value="${druid.maxPoolPreparedStatementPerConnectionSize}" />

		<!-- 监控统计拦截的filters，去掉后监控界面SQL无法统计，https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatFilter 
			http://www.cnblogs.com/ansn001/p/4571606.html -->
		<property name="filters" value="${druid.filters}" />
		<!-- 同时配置filters、proxyFilters时，它们是组合关系并非替换关系 -->
		<property name="proxyFilters">
			<list>
				<ref bean="stat-filter" />
				<ref bean="log-filter" />
			</list>
		</property>
	</bean>

	<!-- Druid内置提供一个StatFilter，用于统计监控信息 -->
	<bean id="stat-filter" class="com.alibaba.druid.filter.stat.StatFilter">
		<!-- slowSqlMillis属性用来配置SQL慢的标准，执行时间超过slowSqlMillis的就是慢，默认值：3000毫秒 -->
		<property name="slowSqlMillis" value="10000" />
		<!-- 通过日志输出执行慢的SQL，默认值：false -->
		<property name="logSlowSql" value="true" />
		<!-- StatFilter提供SQL统计合并的功能，默认值：false -->
		<property name="mergeSql" value="true" />
	</bean>

	<!-- 输出SQL至日志 -->
	<bean id="log-filter" class="com.alibaba.druid.filter.logging.Slf4jLogFilter">
		<!-- 是否输出可执行语句，默认值：false -->
		<property name="statementExecutableSqlLogEnable" value="false" />
	</bean>
</beans>
```

## 配置DruidStatInterceptor

[druid提供了`spring`和`jdbc`的关联监控](https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_Druid%E5%92%8CSpring%E5%85%B3%E8%81%94%E7%9B%91%E6%8E%A7%E9%85%8D%E7%BD%AE)，`com.alibaba.druid.support.spring.stat.DruidStatInterceptor`是一个标准的`spring` `MethodInterceptor`，可以灵活进行aop配置，此处使用方法名正则匹配拦截配置，按类型拦截配置参见wiki。
<br>
<br>
spring aop的配置文档：[http://static.springsource.org/spring/docs/current/spring-framework-reference/html/aop-api.html](http://static.springsource.org/spring/docs/current/spring-framework-reference/html/aop-api.html)

spring-datasource-druid.xml
```xml
<!-- Druid和Spring关联监控配置，方法名正则匹配拦截配置，https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_Druid%E5%92%8CSpring%E5%85%B3%E8%81%94%E7%9B%91%E6%8E%A7%E9%85%8D%E7%BD%AE -->
<bean id="druid-stat-interceptor"
	class="com.alibaba.druid.support.spring.stat.DruidStatInterceptor">
</bean>

<bean id="druid-stat-pointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut"
	scope="prototype">
	<property name="patterns">
		<list>
			<value>com.song.hospital.service.impl.*ServiceImpl</value>
		</list>
	</property>
</bean>

<aop:config>
	<aop:advisor advice-ref="druid-stat-interceptor"
		pointcut-ref="druid-stat-pointcut" />
</aop:config>
```

* [本文所用的完整项目代码托管在github](https://github.com/soyuone/hospital)
* [参考：使用druid监控sql执行](http://blog.csdn.net/flyfish778/article/details/53470683)
* [github：druid](https://github.com/alibaba/druid)
* [wiki：druid常见问题](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)
