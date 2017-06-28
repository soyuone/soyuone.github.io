---
layout: post
title: "常见面试题"
categories: java
tags: java 面试
author: Kopite
---

* content
{:toc}


整理后端常见的面试题，加深对特定知识点的理解。



## 网络协议

* 阐述http协议？
<br>
答：[参见：http协议](https://soyuone.github.io/2017/05/22/network-protocol-http/)

* 阐述TCP连接的三次握手过程？
<br>
答：[参见：TCP/IP协议](https://soyuone.github.io/2017/06/02/network-protocol-tcp-ip/)

* 阐述RESTful架构？
<br>
答：spring mvc原生态的支持REST风格的架构设计，涉及的注解：`@RequestMapping`、`@ResponseBody`、`@PathVariable`等。[参见：RESTful API设计指南](https://soyuone.github.io/2017/06/02/restful-design-api/)

## java基础

* 阐述java异常处理机制？
<br>
答：[参见：java异常处理机制](https://soyuone.github.io/2017/05/14/java-exception/)

* 阐述cookie和session机制？
<br>
答：[参见：cookie和session机制](https://soyuone.github.io/2017/05/23/java-session-cookie/)

* StringBuffer和StringBuilder的区别？
<br>
答：在单线程环境下应该使用StringBuilder来保证较好的性能，当需要保证多线程安全时，就应该使用StringBuffer。

* HashMap和Hashtable的区别？
<br>
答：HashMap和Hashtable都是Map接口的实现类，区别如下：
  * Hashtable是一个线程安全的Map实现，而HashMap是线程不安全的Map实现，所以HashMap比Hashtable的性能高一点；但如果有多个线程访问同一个Map对象时，使用Hashtable实现类会更好
  * Hashtable不允许使用null作为key和value，如果试图把null值放进Hashtable中，将会引发空指针异常；但HashMap可以使用null作为key或value
  * 多线程编程时用ConcurrentHashMap代替Hashtable，因为Hashtable使用的是synchronized关键字，锁住的是对象整体-->[java集合——HashMap、HashTable以及ConCurrentHashMap异同比较](http://blog.csdn.net/seu_calvin/article/details/52653711)

## spring mvc

* 阐述servlet生命周期？servlet什么时候调用destroy()方法？
<br>
答：[参见：理解servlet](https://soyuone.github.io/2017/06/02/java-servlet/)

* @ResponseBody注解的作用？返回的json格式在何处指定？
<br>
答：@ResponseBody注解类位于`spring-web-4.2.0.RELEASE-sources.jar`，其描述如下：

```
Annotation that indicates a method return value should be bound to the web response body. Supported for annotated handler methods in Servlet environments. 

As of version 4.0 this annotation can also be added on the type level in which case it is inherited and does not need to be added on the method level.
```

该注解用于将Controller中方法返回的数据，通过适当的`HttpMessageConverter`转换为指定格式后，写入到该http请求的response body中。常用的一种`HttpMessageConverter`：`MappingJackson2HttpMessageConverter`，其描述如下：

```
Implementation of HttpMessageConverter that can read and write JSON using Jackson 2.x's ObjectMapper. 
This converter can be used to bind to typed beans, or untyped HashMap instances. 
By default, this converter supports application/json and application/*+json. This can be overridden by setting the supportedMediaTypes property. 
The default constructor uses the default configuration provided by Jackson2ObjectMapperBuilder. 
Compatible with Jackson 2.1 and higher. 
```

## spring framework



## database

* 阐述共享锁、排它锁，表锁、行锁，ACID，隔离级别，乐观锁、悲观锁？
<br>
答：[参见：mysql架构与历史](https://soyuone.github.io/2017/06/06/database-mysql-high-performance/)

* 数据库设计时数据类型的选择？
<br>
答：[参见：schema与数据类型优化](https://soyuone.github.io/2017/06/07/database-mysql-high-performance-data-type/)

## redis


