---
layout: post
title: "http协议"
categories: 网络协议
tags: 网络协议 http
author: Kopite
---

* content
{:toc}


客户端与服务器交互时需要用到`http协议`，本文进行整理。 



## 简介

`http协议`（HyperText Transfer Protocol，超文本传输协议）是因特网上应用最为广泛的一种网络传输协议。`http协议`是基于`TCP/IP协议`的应用层协议，它不涉及数据包（packet）传输，主要规定了客户端和服务器之间的通信格式，默认使用80端口。

### http/0.9

#### 请求格式

http/0.9只有一个命令`GET`，如下示例中，TCP连接（connection）建立后，客户端向服务器请求（request）网页`index.html`。

```
GET /index.html
```

#### 回应格式

http/0.9版本规定服务器只能回应html格式的字符串，服务器发送完毕就关闭TCP连接。

```
<html>
  <body>Hello World</body>
</html>
```

### http/1.0

在1996年5月发布的HTTP/1.0版本中：
* 可以发送任何格式的内容，这使互联网从此不仅可以传输文字，还可以传输图像、视频以及二进制文件
* 除了已有的`GET`命令，HTTP/1.0还引入了`POST`、`HEAD`命令，丰富了浏览器与服务器的交互方式
* http请求和回应的格式也有所改变，除了数据部分，每次通信都必须包括头信息（http header），它被用来描述一些元数据
* 其他新增的功能还包括状态码（status code）、多字符集支持、多部分发送（multi-part type）、权限（authorization）、缓存（cache）、内容编码（content encoding）等

#### 请求格式

```
GET / HTTP/1.0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5)
Accept: */*
```

上述请求示例代码中，第一行是请求行，必须在尾部添加协议版本`HTTP/1.0`，后面是请求头信息，用来描述客户端的情况。
* Accept —— 声明客户端可以接受的数据类型，`Accept: */*`表明客户端可以接受任意类型的数据
* Accept-Encoding —— 声明客户端可以接受的数据压缩方法，如gzip、deflate等

#### 回应格式

服务器回应如下所示，回应的格式：`响应行 + 响应头部 + 空行 + 响应数据`，其中`响应行`由`协议版本 + 状态码（status code） + 状态描述`组成。

```
HTTP/1.0 200 OK 
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 05 Dec 1997 16:00:00 GMT
Last-Modified: Wed, 5 August 1996 15:55:28 GMT
Server: Apache 0.84

<html>
  <body>Hello World</body>
</html>
```

* Content-Type —— 服务器回应的数据类型，这些数据类型称为[MIME（Multipurpose Internet Mail Extensions）](http://www.w3school.com.cn/media/media_mimeref.asp)。可以在`MIME`尾部使用分号用来附加参数，如下所示，服务器回应的是网页并且编码方式为utf-8
```
Content-Type: text/html; charset=utf-8
```  
* Content-Encoding —— 由于服务器回应的数据类型可以任意，`Content-Encoding`指定服务器所回应数据的压缩方法，如gzip、compress、deflate等

#### 缺点

http/1.0版本的缺点在于每个TCP连接只能发送一个请求，数据发送完毕，连接就关闭，如果还要请求其他资源，就需要再新建一个连接，而TCP连接的新建成本很高，因为客户端和服务器需要三次握手，并且开始时发送速率较慢（slow start），http/1.0版本性能较差，随着网页加载的外部资源越来越多，这个问题愈发突出。
<br>
<br>
为解决此问题，有些浏览器在请求时，使用了一个非标准的`Connection`属性，如下所示。

``` 
Connection: keep-alive
``` 

这个属性要求服务器不关闭TCP连接，以便供其他请求复用，服务器同样回应这个属性，如下所示。

``` 
Connection: keep-alive
``` 

一个可以复用的TCP连接就建立了，直到客户端或服务器主动关闭连接，但是`Connection`并不是标准属性。

### http/1.1

#### 持久连接

http/1.1版本于1997年1月发布，它进一步完善了`http协议`。该版本引入了`持久连接（persistent connection）`，即TCP连接默认不关闭，可以被多个请求复用，不需要声明`Connection: keep-alive`。
<br>
<br>
当客户端和服务器发现对方有一段时间没有活动，就可以主动关闭连接，相对规范的做法：客户端在最后一个请求时发送`Connection: close`，明确要求服务器关闭TCP连接。

``` 
Connection: close
``` 

#### 管道机制

http/1.1版本还引入了`管道机制（pipelining）`，即在同一个TCP连接里面，客户端可以同时发送多个请求，提升`http协议`的效率。例如，客户端需要请求两个资源时，以前的做法是在同一个TCP连接里面，先发送A请求，然后等待服务器做出回应，客户端收到服务器回应后再发出B请求。http/1.1版本引入的`管道机制`允许浏览器同时发出A请求和B请求，但是服务器还是按照顺序，先回应A请求，完成后再回应B请求。
<br>
<br>
一个TCP连接现在可以传送多个回应，势必就要有一种机制来区分数据包是属于哪一个回应，Content-length属性的作用：声明本次回应的数据长度。

``` 
Content-Length: 3495
``` 

上面示例中告诉浏览器，服务器本次回应的长度是3495个字节，后面的字节属于下一个回应。在http/1.0版本中，`Content-Length`属性不是必需，因为浏览器发现服务器关闭TCP连接时，就表明浏览器已经收到全部数据包。
<br>
<br>
使用`Content-Length`属性的前提条件是：服务器发送回应之前，必须知道回应的数据长度。对一些很耗时的动态操作来说，这意味着服务器要等到所有操作完成才能发送数据，导致效率并不高。

#### 分块传输编码

http/1.1版本引入了`分块传输编码（chunked transfer encoding）`，产生一块数据就发送一块，采用流模式（stream）取代缓存模式（buffer）。
<br>
<br>
因此，http/1.1版本规定可以不使用`Content-Length`属性，转而使用`分块传输编码`，只要请求或回应的头信息有`Transfer-Encoding`属性，就表明回应将由数量未定的数据块组成。

``` 
Transfer-Encoding: chunked
``` 

每个非空的数据块之前，会有一个16进制的数值表示这个块的长度，最后一个大小为0的块表示本次回应的数据发送完毕，示例如下

``` 
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

25
This is the data in the first chunk

1C
and this is the second one

3
con

8
sequence

0
``` 

#### 其他功能

* http/1.1新增多个命令：`PUT`、`PATCH`、`HEAD`、`OPTIONS`、`DELETE`
* 客户端请求头信息中新增`Host`属性，该属性指定服务器的域名，如下所示。

``` 
Host: www.example.com
``` 

#### 缺点

虽然http/1.1允许复用TCP连接，但是在同一个TCP连接里面，所有的数据通信是按次序进行的，服务器只有处理完一个回应才会进行下一个回应，一旦前面的回应特别慢，后面就会有许多请求排队等待，这称为`队头堵塞（Head-of-line blocking）`。
<br>
<br>
为避免此问题，只有两种方法：一是减少请求数，二是同时多开持久连接，由此产生很多网页优化技巧，比如合并脚本和样式表、将图片嵌入css代码、域名分片（domain sharding）等。

### http/2

http/2于2015年发布，它由google开发的`SPDY协议`演化而来。[http协议简介部分系转载，详见：http协议入门](http://www.ruanyifeng.com/blog/2016/08/http.html)

## 原理

`http协议`工作于C/S、B/S架构，客户端通过URL向服务器发送请求，web服务器根据接收到的请求，向客户端发送回应信息，默认使用80端口。
<br>
<br>
`http协议`定义了客户端如何向服务器请求数据，以及服务器如何把响应数据传送给客户端。客户端向服务器发送一个请求报文，请求报文包含请求的方法、URL、协议版本、请求头部和请求数据，服务器以一个状态行作为响应，响应的内容包括协议的版本、成功或者错误代码、服务器信息、响应头部和响应数据。

![](/image/2017/2017-05-22-network-protocol-http-1.jpg)

http请求/响应的步骤：
* 客户端连接到服务器。客户端与服务器的端口（默认为80）建立一个TCP套接字连接，例如`http://www.oakcms.cn`
* 客户端发送http请求。通过TCP套接字，客户端向服务器发送请求报文，一个请求报文由`请求行`、`请求头部`、`空行`和`请求数据`4部分组成
* 服务器接受请求并返回http响应。服务器解析请求，定位请求资源，并将资源复本写到TCP套接字，由客户端读取，一个响应由`响应行`、`响应头部`、`空行`和`响应数据`4部分组成
* 释放TCP连接。如果`Connection`属性为close，服务器主动关闭TCP连接，客户端被动关闭连接，释放TCP连接；如果`Connection`属性为keep-alive，则该连接会保持一段时间，在该时间内可以继续接收请求
* 客户端解析html内容。客户端首先解析状态行，查看请求状态码，然后解析响应头

在浏览器地址栏输入URL，按下回车键后的请求响应流程：
1. 浏览器向DNS服务器请求解析该URL中域名所对应的ip地址
2. 解析出ip地址后，根据该ip地址和端口同服务器建立TCP连接
3. 浏览器发出读取文件（URL中域名后面部分对应的文件）的http请求，该请求报文作为TCP三次握手中第三个报文的数据发送给服务器
4. 服务器对浏览器请求作出响应，并把对应的html文本发送给浏览器
5. 释放TCP连接
6. 浏览器接收html文本并显示内容  

`http协议`的主要特点如下：
* 媒体独立。任意类型的数据都可以通过`http协议`发送，客户端及服务器指定合适的`MIME`内容类型即可
* 无状态。无状态是指`http协议`对事务处理没有记忆能力，如果后续处理需要前面的信息时，需要进行重传，这将导致每次连接传送的数据量变大

客户端请求消息格式由`请求行（request line）`、`请求头部（request header）`、`空行`和`请求数据（request body）`四部分组成，如下图所示。

![](/image/2017/2017-05-22-network-protocol-http-2.png)

* METHOD表示请求类型，例如`POST`、`GET`
* path-to-resoure表示请求的资源
* HTTP/Version-number表示http协议的版本号
* `GET`请求时`request body`部分为空

服务器响应也由四部分组成：`响应行（response line）`、`响应头部（response header）`、`空行`和`响应数据（response body）`，如下图所示。

![](/image/2017/2017-05-22-network-protocol-http-3.png)

## URI与URL

URI（Uniform Resource Identifier，统一资源标识符）用来唯一的标识一个资源，web上可用的每种资源如html文档、图像、视频片段、程序等都使用URI来定位。
<br>
<br>
URI一般由三部组成：`访问资源的命名机制`、`存放资源的主机名`、`资源自身的名称（由路径表示）`。
<br>
<br>
URL（Uniform Resource Locator，统一资源定位符）是一种具体的URI。笼统的说，每个URL都是URI，但每个URI不一定都是URL，这是因为URI还包括URN。URL用来标识某一处资源的地址，以下述URL为例来介绍URL的各部分组成。

``` 
http://www.aspxfans.com:8080/news/index.asp?boardID=5&ID=24618&page=1#name
``` 

* 协议部分，该URL的协议部分为`http：`，表示使用了`http协议`，在互联网中可以使用多种协议，如http、ftp等，后面的`//`为分隔符
* 域名部分，该URL的域名部分为`www.aspxfans.com`，也可以使用ip地址作为域名
* 端口部分，跟在域名后面的是端口，域名和端口之间使用`:`作为分隔符，端口不是必须，省略端口时将使用默认端口
* 虚拟目录部分，从域名后的第一个`/`开始到最后一个`/`止是虚拟目录部分，虚拟目录也不是URL中必须的部分。本例中的虚拟目录是`/news/`
* 文件名部分，从域名后的最后一个`/`开始到`？`止是文件名部分；如果没有`?`，则从域名后的最后一个`/`开始到`#`止是文件名部分；如果没有`？`和`#`，那么从域名后的最后一个`/`开始到URL结束都是文件名部分，文件名不是URL中必须的部分，省略时将使用默认的文件名。本例中的文件名是`index.asp`
* 参数部分，从`？`开始到`#`止是参数部分，又称搜索部分、查询部分，允许有多个参数，多个参数之间用`&`分隔。本例中的参数部分是`boardID=5&ID=24618&page=1`
* 锚部分，从`#`开始到最后是锚部分，锚不是URL中必须的部分。本例中的锚部分是`name`

## 状态码

访问网页时，浏览器会向服务器发出请求，在浏览器接收并显示网页前，服务器会返回一个包含http状态码（HTTP Status Code）的信息头（server header）用以响应浏览器的请求。
<br>
<br>
http状态码由三个十进制数字组成，第一个十进制数字定义了状态码的类型，后两个数字没有分类的作用，http状态码共分为5种类型，如下图所示。

![](/image/2017/2017-05-22-network-protocol-http-4.png)

常见的状态码如下：
* 200 - 请求成功
* 301 - 资源（网页等）被永久转移到其它URL
* 404 - 请求的资源（网页等）不存在
* 500 - 内部服务器错误

## GET与POST请求的区别

### GET示例

chrome浏览器，[博客园首页](https://www.cnblogs.com/)搜索框中输入java，使用[Fiddler工具](http://www.telerik.com/fiddler)查看Request、Response信息，如下所示：

URL
```
https://www.cnblogs.com/
```
<br>
request
```
GET http://zzk.cnblogs.com/s?t=b&w=java HTTP/1.1
Host: zzk.cnblogs.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.2; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Cookie: __utma=226521935.1751255329.1471419249.1475723920.1475723920.1; __utmt=1; __utma=59123430.1751255329.1471419249.1498449233.1498449233.1; __utmb=59123430.1.10.1498449233; __utmc=59123430; __utmz=59123430.1498449233.1.1.utmcsr=(direct)|utmccn=(direct)|utmcmd=(none); _ga=GA1.2.1751255329.1471419249; _gid=GA1.2.1232116599.1498397326; _gat=1
```
<br>
response
```
HTTP/1.1 200 OK
Date: Mon, 26 Jun 2017 03:55:01 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Vary: Accept-Encoding
Cache-Control: private
X-AspNetMvc-Version: 5.2
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Content-Length: 28192

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title> java - 博客园找找看</title>
    <link rel="shortcut icon" href="/Content/Images/favicon.ico" type="image/x-icon" />
    <link rel="stylesheet" href="//at.alicdn.com/t/font_1476446892_1265016.css"/>
    <link href="/bundles/vendor/css/search?v=TPZ2g11WKxuQMoDW28Ba86EiiXG-IC754vBdoWpS_ck1" rel="stylesheet"/>

    <link href="/bundles/css/common?v=YnyBIbVcjYfvSniBrK1MruBuU3tVNDUW8efOvdBtlzA1" rel="stylesheet"/>

    <link href="/bundles/css/search?v=7Idlz8AusWLK0z-1fKUFATCMjb4f-bYnikvDTCoFLm41" rel="stylesheet"/>

    <script src="//common.cnblogs.com/script/jquery.js" type="text/javascript"></script>
    <script src="/bundles/vendor/js/search?v=NpLlfw4LpMCUZhjzyVCso1691Ad_n968KGHO0lB-39o1"></script>

    <script src="/bundles/js/common?v=ySabubpZbwHmGc1mYw-D1HcILTcuF3A5bYjm7gz9Ilg1"></script>

    <script src="/bundles/js/search?v=gJDH3XYMp1k0TzoPyxL-VTFkjSL1aEkZGvvVPe4qeyg1"></script>

</head>
<body>
         ...
</body>
</html>
```

### POST示例

chrome浏览器，[hospital项目登录页](https://github.com/soyuone/hospital)中输入账号及密码，使用[Fiddler工具](http://www.telerik.com/fiddler)查看Request、Response信息，如下所示：

URL
```
localhost:8080/hospital/login.jsp
```
<br>
request
```
POST http://localhost:8080/hospital/api/user/login HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 38
Accept: application/json, text/javascript, */*; q=0.01
Origin: http://localhost:8080
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 6.2; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Referer: http://localhost:8080/hospital/login.jsp
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Cookie: SESSION=71c064a9-a420-4173-81ad-ed928445574b; Hm_lvt_9519c406f5a0c72b10a515c2cea174f9=1498303263,1498351236,1498375768,1498441810; Hm_lpvt_9519c406f5a0c72b10a515c2cea174f9=1498449901

email=soyuone%40sina.cn&password=admin
```
<br>
response
```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Mon, 26 Jun 2017 04:10:51 GMT

21
{"code":200,"msg":"登录成功"}
0
```

### 区别

* GET请求的参数附在URL之后；POST请求的参数放在request body中传输，URL并不会改变
* `http协议`没有对传输数据的大小进行限制，但在实际开发中存在限制。GET请求传输的数据量较小，一般不能大于2KB；POST请求传送的数据量较大，通常认为POST请求参数的大小不受限制，但实际上web服务器会对POST请求时的数据大小进行限制，POST请求传输的数据量总比GET请求传输的数据量大
* POST请求的安全性比GET请求的安全性高

* [参考：http协议详解](http://www.cnblogs.com/TankXiao/archive/2012/02/13/2342672.html)
* [参考：http协议入门](http://www.ruanyifeng.com/blog/2016/08/http.html)
* [参考：https升级指南](http://www.ruanyifeng.com/blog/2016/08/migrate-from-http-to-https.html)
* [参考：http协议小结](http://www.cnblogs.com/ranyonsue/p/5984001.html)
* [参考：http协议-菜鸟教程](http://www.runoob.com/http/http-tutorial.html)
