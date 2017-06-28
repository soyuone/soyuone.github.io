---
layout: post
title: "理解servlet"
categories: java
tags: java servlet
author: Kopite
---

* content
{:toc}


`jsp（java server page）`和`servlet`是java ee规范的两个基本成员，它们的本质是一样的，因为`jsp`最终必须编译成`servlet`才能运行。`jsp`、`servlet`、`listener`和`filter`等都必须运行在web应用中。
<br>
<br>
`jsp`的特点是在html页面中嵌入java代码片段，或使用各种`jsp`标签，包括用户自定义标签，从而可以动态的提供页面内容。自java ee标准出现后，`jsp`慢慢的发展成单一的表现层技术，不再承担业务逻辑组件及持久层组件的责任。`freemarker`、`velocity`、`tapestry`等表现层技术基本可以取代jsp技术，但`jsp`依然是应用最广范的表现层技术。



## 配置描述符

每个web应用的`\WEB-INF\web.xml`文件被称为`配置描述符`，在`servlet 2.5`规范之前，每个java web应用都必须在上述路径下包含一个`web.xml`文件；从`servlet 3.0`规范开始，`\WEB-INF\web.xml`文件不再是必需，可以通过注解来配置管理web组件，但通常还是保留该配置文件。
<br>
<br>
`web.xml`文件的根元素是`<web-app.../>`元素，在`servlet 3.0`规范中，该元素中新增了`metadata-complete`属性，该属性接受`true/false`值，当值为`true`时，web应用将不会加载注解配置的web组件，如`servlet`、`filter`、`listener`等。
<br>
<br>
在`web.xml`文件中配置首页使用`<welcome-file-list>`元素，该元素能包含多个`<welcome-file>`子元素，其中每个`<welcome-file>`子元素配置一个首页，如下所示。

```xml
<!-- 配置首页，依次寻找 -->
<welcome-file-list>
	<welcome-file>login.jsp</welcome-file>
	<welcome-file>index.jsp</welcome-file>
	<welcome-file>index.htm</welcome-file>
	<welcome-file>index.html</welcome-file>
</welcome-file-list>
```

## jsp的基本原理

jsp页面的内容由如下两部分组成：

* 静态部分，标准的html标签、静态的页面内容，这些内容与静态html页面相同
* 动态部分，受java程序控制的内容，这些内容由java脚本动态生成

`jsp`页面的本质是`servlet`，每个`jsp`页面都是一个`servlet`实例——`jsp`页面由`servlet`容器生成对应的`servlet`，`servlet`再负责响应用户请求。对于tomcat而言，`jsp`页面生成的`servlet`放在`\work\`路径对应的web应用下，示例如下。

test.jsp
```xml
<%@ page language="java" contentType="text/html; charset=UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>查看servlet</title>
</head>
<body>

	<%-- jsp注释格式 --%>
	<!-- html注释格式 -->
	<!-- jsp文件必须在jsp服务器内运行；jsp文件必须生成servlet才能执行；每个jsp页面的第一个访问者速度较慢，要等待编译为servlet -->
	<!-- jsp页面输送到客户端的是标准html页面 -->

	<!-- <font>规定文本的字体、字体尺寸、字体颜 -->
	<!-- <br>为换行符 -->
	
	<!-- tomcat根据jsp页面生成对应servlet的java文件、class文件 -->
	<%
		for (int i = 0; i < 7; i++) {
			out.println("<font size='" + i + "'>");
	%>
	疯狂Java训练营(Wild Java Camp)</font>
	<br/>
	<%}%>
</body>
</html>
```

当tomcat启动后，可以在`\work\Catalina\localhost\jspservlet\org\apache\jsp\`路径下找到`test_jsp.java`、`test_jsp.class`，这两个文件都是由tomcat生成，tomcat根据`jsp`页面生成对应的java文件和class文件，浏览器中页面显示如下。

![](/image/2017/2017-06-02-java-servlet-1.png)

`test_jsp.java`是一个java类，`jsp`页面中的所有内容都由该类的页面输出流完成，`test_jsp`类主要包含如下三个方法：

* _jspInit()，初始化`jsp/servlet`的方法
* _jspDestroy()，销毁`jsp/servlet`的方法
* _jspService()，对用户请求生成响应的方法

`jsp`页面的工作原理如下图所示：

![](/image/2017/2017-06-02-java-servlet-2.png)

## servlet

`servlet`通常称为`服务器端小程序`，是运行在服务器端的程序，用于处理及响应客户端的请求。mvc规范出现后，`servlet`通常仅作为控制器使用，不再用于表现层，作用类似于调度员：所有请求都发送给`servlet`，`servlet`调用`Model`来处理用户请求，并调用`jsp`来呈现处理结果，或者`servlet`直接调用`jsp`将应用的状态数据呈现给用户。

### 开发

`servlet`是个特殊的java类，必须继承`javax.servlet.http.HttpServlet`类。每个`servlet`可以响应客户端的请求，`servlet`提供不同的方法用于响应客户端请求：
* doGet()，用于响应客户端的GET请求
* doPost()，用于响应客户端的POST请求
* doPut()，用于响应客户端的PUT请求
* doDelete()，用于响应客户端的DELETE请求

事实上，客户端的请求通常只有GET、POST两种，`servlet`为了响应这两种请求，必须重写doGet()、doPost()方法。大多数情况下，`servlet`对于所有请求的响应都是完全一样的，此时可以重写一个方法来代替上面的几个方法：只需要重写service()方法即可响应客户端的所有请求。
<br>
<br>
`javax.servlet.http.HttpServlet`类还包含如下两个方法：
* init()，创建servlet实例时，调用该方法以初始化资源
* destroy()，销毁servlet实例时，自动调用该方法以回收资源

通常无须重写init()、destroy()方法，除非在初始化`servlet`时，需要完成对某些资源初始化，才重写init()方法。如果在销毁`servlet`之前，需要先完成某些资源的回收，比如关闭数据库连接等，才重写destroy()方法，示例如下。

FirstServlet.java
```java
@SuppressWarnings("serial")
public class FirstServlet extends HttpServlet {

	@Override
	public void destroy() {
		super.destroy();
		System.out.println("--------FirstServlet destroy方法--------");
	}

	@Override
	public void init() throws ServletException {
		super.init();
		System.out.println("--------FirstServlet init方法--------");
	}

	// 客户端的响应方法，使用该方法可以响应客户端所有类型的请求
	@Override
	public void service(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, java.io.IOException {
		System.out.println("--------FirstServlet service方法--------");
		// 设置解码方式
		request.setCharacterEncoding("UTF-8");
		response.setContentType("text/html;charSet=UTF-8");
		// 获取name的请求参数值
		String name = request.getParameter("name");
		// 获取gender的请求参数值
		String gender = request.getParameter("gender");
		// 获取color的请求参数值
		String[] color = request.getParameterValues("color");
		// 获取country的请求参数值
		String national = request.getParameter("country");
		// 获取页面输出流
		PrintStream out = new PrintStream(response.getOutputStream());
		// 输出HTML页面标签
		out.println("<html>");
		out.println("<head>");
		out.println("<title>Servlet测试</title>");
		out.println("</head>");
		out.println("<body>");
		// 输出请求参数的值：name
		out.println("您的名字：" + name + "<hr/>");
		// 输出请求参数的值：gender
		out.println("您的性别：" + gender + "<hr/>");
		// 输出请求参数的值：color
		out.println("您喜欢的颜色：");
		for (String c : color) {
			out.println(c + " ");
		}
		out.println("<hr/>");
		out.println("您喜欢的颜色：");
		// 输出请求参数的值：national
		out.println("您来自的国家：" + national + "<hr/>");
		out.println("</body>");
		out.println("</html>");
	}
}
```

普通`servlet`类里的service()方法的作用，完全等同于`jsp`生成`servlet`类的_jspService()方法。因此`jsp`页面的`jsp`脚本、静态html内容，在普通`servlet`中都应该转换成service()方法的代码或输出语句；`jsp`声明中的内容，对应`servlet`中定义的成员变量或成员方法。`servlet`和`jsp`的区别在于：
* `servlet`中没有内置对象，原来`jsp`中的内置对象都必须由程序显式创建
* 对于静态的html标签，`servlet`都必须使用页面输出流逐行输出

### 配置

从`servlet 3.0`开始，配置`servlet`有两种方式：
1. 在`servlet`类中使用@WebServlet注解进行配置
2. 在web.xml文件中进行配置

如果使用注解配置`servlet`：
* 不要在web.xml文件的根元素中指定`metadata-complete="true"`
* 不要在web.xml文件中配置该`servlet`

如果使用web.xml配置`servlet`，示例如下：

```xml
<servlet>
	<servlet-name>firstServlet</servlet-name>
	<servlet-class>com.song.jspservlet.FirstServlet</servlet-class>
</servlet>

<servlet-mapping>
	<servlet-name>firstServlet</servlet-name>
	<url-pattern>/firstServlet</url-pattern>
</servlet-mapping>
```

* 配置`servlet`的名字
* 配置`servlet`的URL，如果没有为`servlet`配置URL，则该`servlet`不能响应用户请求

### 生命周期

`jsp`页面的本质是`servlet`，页面将由web容器编译成对应的`servlet`，`servlet`在容器中运行时，其实例的创建及销毁由web容器进行控制，创建`servlet`实例有两个时机：
1. 客户端第一次请求某个`servlet`时，系统创建该`servlet`的实例，大多数`servlet`都属这种
2. web应用启动时立即创建`servlet`实例，即`load-on-startup``servlet` 

每个`servlet`的运行都遵循如下生命周期：
1. 创建`servlet`实例
2. web容器调用`servlet`的init()方法，对`servlet`进行初始化
3. `servlet`初始化后，将一直存在于容器中，用于响应客户端请求。如果客户端发送GET请求，容器调用`servlet`的doGet方法处理并响应请求；如果客户端发送POST请求，容器调用`servlet`的doPost()方法处理并响应请求；或者统一使用service()方法响应请求
4. web容器决定销毁`servlet`时，先调用`servlet`的destroy()方法，通常在关闭web应用时销毁`servlet`

`servlet`的生命周期如下图所示：

![](/image/2017/2017-06-02-java-servlet-3.png)

#### 示例一

客户端第一次请求某个`servlet`时，系统创建`servlet`实例。web应用启动后，chrome浏览器中输入如下URL：

```
localhost:8080/jspservlet/firstServlet?name=songyu&gender=male&color=red&country=china
```

控制台输出：
```
信息: Server startup in 1737 ms
--------FirstServlet init方法--------
--------FirstServlet service方法--------
```

chrome浏览器中再次输入如下URL，并改变部分参数的值：

```
http://localhost:8080/jspservlet/firstServlet?name=songyu&gender=male&color=green&country=japan
```

控制台输出：
```
--------FirstServlet service方法--------
```

关闭web应用，控制台输出如下：

```
--------FirstServlet destroy方法--------
```

小结如下：
1. 客户端在第一次请求`servlet`时，调用init()方法对`servlet`进行初始化
2. 调用service()方法响应请求
3. 当第二次、第三次···请求该`servlet`时，不再调用init()方法，直接调用service()方法响应请求
4. 当关闭web应用时，调用destroy()方法回收资源

#### 示例二

web应用启动时立即创建`servlet`实例，添加`<load-on-startup>1</load-on-startup>`，`<load-on-startup>`中属性值越小越优先实例化，应用启动后控制台输出：
```
--------FirstServlet init方法--------
六月 08, 2017 12:42:02 上午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["http-apr-8080"]
六月 08, 2017 12:42:02 上午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["ajp-apr-8009"]
六月 08, 2017 12:42:02 上午 org.apache.catalina.startup.Catalina start
信息: Server startup in 1682 ms
```

chrome浏览器中输入如下URL：

```
localhost:8080/jspservlet/firstServlet?name=songyu&gender=male&color=red&country=china
```

控制台输出：
```
--------FirstServlet service方法--------
```

chrome浏览器中再次输入如下URL，并改变部分参数的值：

```
http://localhost:8080/jspservlet/firstServlet?name=songyu&gender=male&color=green&country=japan
```

控制台输出：
```
--------FirstServlet service方法--------
```

关闭web应用，控制台输出如下：

```
--------FirstServlet destroy方法--------
```

小结如下：
1. web应用启动时调用init()方法对`servlet`进行初始化
2. 当第一次、第二次···请求该`servlet`时，不再调用init()方法，直接调用service()方法响应请求
3. 当关闭web应用时，调用destroy()方法回收资源

* [本文所用的完整项目代码托管在github](https://github.com/soyuone/jspservlet)
* [参考：《轻量级java ee企业应用实战》]()