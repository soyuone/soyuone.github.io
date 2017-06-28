---
layout: post
title: "理解RESTful"
categories: RESTful
tags: RESTful
author: Kopite
---

* content
{:toc}

本文系转载，原文：[阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/09/restful)
<br>
<br>
`RESTful`是一种互联网软件架构，核心是面向资源，它降低了开发的复杂性、提高了系统的可伸缩性。



## 起源

REST由[Roy Thomas Fielding](https://en.wikipedia.org/wiki/Roy_Fielding)在他2000年的[博士论文](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)中提出，这篇论文一经发表就引起关注，对互联网开发产生了深远的影响，他这样介绍论文的写作目的：

```
"本文研究计算机科学两大前沿——软件和网络——的交叉点。长期以来，软件研究主要关注软件设计的分类、设计方法的演化，很少客观的评估不同的设计选择对系统行为的影响。而相反的，网络研究主要关注系统之间通信行为的细节、如何改进特定通信机制的表现，常常忽视了一个事实，那就是改变应用程序的互动风格比改变互动协议，对整体表现有更大的影响。我这篇文章的写作目的，就是想在符合架构原理的前提下，理解和评估以网络为基础的应用软件的架构设计，得到一个功能强、性能好、适宜通信的架构。"

(This dissertation explores a junction on the frontiers of two research disciplines in computer science: software and networking. Software research has long been concerned with the categorization of software designs and the development of design methodologies, but has rarely been able to objectively evaluate the impact of various design choices on system behavior. Networking research, in contrast, is focused on the details of generic communication behavior between systems and improving the performance of particular communication techniques, often ignoring the fact that changing the interaction style of an application can have more impact on performance than the communication protocols used for that interaction. My work is motivated by the desire to understand and evaluate the architectural design of network-based application software through principled use of architectural constraints, thereby obtaining the functional, performance, and social properties desired of an architecture. )
```

## 名称

Roy Thomas Fielding将他对互联网软件的架构原则，命名为`REST（Representational State Transfer）`。如果一个架构符合`REST原则`，就称它为`RESTful架构`。要理解`RESTful架构`，最好的方法就是去理解`Representational State Transfer`这个词组的意思，它的每一个词的含义。

## 资源

`REST`的名称`表现层状态转化`中省略了主语，`表现层`指的是`资源（Resources）`的`表现层`。所谓`资源`，就是网络上的一个实体，或者说是网络上的一个具体信息，可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。可以用一个`URI（统一资源定位符）`指向它，每种资源对应一个特定的`URI`，要获取这个资源，访问它的`URI`就可以，因此`URI`就成了每一个资源的地址或独一无二的识别符。
<br>
<br>
所谓`上网`，就是与互联网上一系列的`资源`互动，调用它的`URI`。

## 表现层

`资源`是一种信息实体，它可以有多种外在表现形式，把`资源`具体呈现出来的形式叫做`资源`的`表现层（Representation）`。例如，文本可以用txt格式，也可以用html格式、xml格式、json格式，甚至可以采用二进制格式；图片可以用jpg格式，也可以用png格式。
<br>
<br>
`URI`只代表`资源`的实体，不代表它的形式。严格地说，有些网址最后的.html后缀名是不必要的，因为这个后缀名表示格式，属于`表现层`范畴，而`URI`应该只代表`资源`的位置。它的具体表现形式，应该在`http`请求的头信息中用`Accept`和`Content-Type`字段指定，这两个字段才是对`表现层`的描述。

## 状态转化

访问一个网站，代表了客户端和服务器的一个互动过程，此过程势必涉及到数据和状态的变化。`http协议`是一个无状态协议，这意味着所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生`状态转化（State Transfer）`，而这种转化是建立在`表现层`之上的，即`表现层状态转化`。
<br>
<br>
客户端用到的手段，只能是`http协议`。具体来说，就是`http协议`中四个表示操作方式的动词：`GET`、`POST`、`PUT`、`DELETE`，它们分别对应四种基本操作：`GET`用来获取资源，`POST`用来新建资源（也可以用于更新资源），`PUT`用来更新资源，`DELETE`用来删除资源。

## 综述

综合上面的解释，总结一下什么是`RESTful架构`：
1. 每一个`URI`代表一种资源
2. 客户端和服务器之间，传递这种资源的某种表现层
3. 客户端通过四个`http`动词，对服务器端资源进行操作，实现`表现层状态转化`

## 误区

`RESTful架构`有一些典型的设计误区：
<br>
<br>
最常见的一种设计错误，就是`URI`包含动词。因为`资源`表示一种实体，所以应该是名词，`URI`不应该有动词，动词应该放在`http协议`中。
<br>
<br>
举例来说，某个`URI`是`/posts/show/1`，其中show是动词，这个`URI`就设计错了，正确的写法应该是`/posts/1`，然后用`GET`方法表示show。
<br>
<br>
如果某些动作是`http`动词表示不了的，此时应该把动作当作一种资源。比如网上汇款，从账户1向账户2汇款500元，错误的`URI`：

```
POST /accounts/1/transfer/500/to/2
```

正确的写法是把动词transfer改成名词transaction，资源不能是动词，但是可以是一种服务：

```
POST /transaction HTTP/1.1
Host: 127.0.0.1
　　
from=1&to=2&amount=500.00
```

另一个设计误区，就是在`URI`中加入版本号：

```
http://www.example.com/app/1.0/foo
http://www.example.com/app/1.1/foo
http://www.example.com/app/2.0/foo
```

因为不同的版本，可以理解成同一种资源的不同表现形式，所以应该使用同一个`URI`，版本号可以在`http`请求头信息的`Accept`字段中进行区分，参见：[Versioning REST Services](http://www.informit.com/articles/article.aspx?p=1566460)：

```
Accept: vnd.example-com.foo+json; version=1.0
Accept: vnd.example-com.foo+json; version=1.1
Accept: vnd.example-com.foo+json; version=2.0
```

* [参考：理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful)
* [参考：RESTful API设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
* [参考：Restful API实战](http://www.imooc.com/learn/811)