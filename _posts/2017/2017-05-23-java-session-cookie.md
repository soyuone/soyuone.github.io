---
layout: post
title: "cookie和session机制"
categories: java
tags: java session cookie http
author: Kopite
---

* content
{:toc}


web应用使用http协议进行客户端与服务器的交互，http协议是无状态的协议，一旦数据交换完毕，客户端与服务器的连接就会关闭，再次交换数据时需要建立新的连接，这导致服务器无法从连接中跟踪会话。例如，用户A购买了一件商品并放入购物车中，当再次购买商品时服务器已经无法判断该购买行为属于用户A还是用户B的会话。
<br>
<br>
常用的会话跟踪技术有cookie、session，其中，cookie通过在客户端记录信息以确定用户身份，session通过在服务器端记录信息以确定用户身份，本文对两种机制进行整理。 



## cookie机制

### 简介

cookie通常也叫做网站cookie、浏览器cookie或http cookie，保存在浏览器端，它可以用来做用户认证、服务器校验等使用文本数据可以处理的问题。cookie机制可以弥补http协议无状态的缺点，在session出现之前，很多网站都采用cookie来跟踪会话。
<br>
<br>
cookie实际是一段文本信息，当客户端请求服务器时，如果服务器需要记录该用户状态，response就向客户端颁发一个cookie，客户端会把cookie保存起来，浏览器再次请求该网站时，客户端连同该cookie一起提交给服务器，服务器检查该cookie以辨认用户状态。

### 类别

* session cookie，这种类型的cookie只在会话期间内有效，当关闭浏览器的时候，它会被浏览器删除。设置session cookie的办法是：创建cookie时不设置`maxAge`属性即可，会话cookie通常存储在内存而非硬盘中
* persistent cookie，持久型cookie会长期在用户会话中生效，通过设置cookie的`maxAge`属性，在接下来的这段时间，相关URL的每次http请求都会携带这个cookie，浏览器通常会把持久型cookie存储到硬盘中

```java
String HOSPITALTOKEN = "HOSPITALTOKEN";

Cookie cookie = new Cookie(HOSPITALTOKEN, CommonUtil.generateId());
// an integer specifying the maximum age of the cookie in seconds;
// if negative, means the cookie is not stored; if zero, deletes the cookie
cookie.setMaxAge(30 * 24 * 60 * 60);
```

* secure cookie，[安全cookie是在`https协议`下的cookie形态，以确保cookie在从客户端传递到服务器的过程中始终加密，这样做降低了cookie内容直接暴露在黑客面前及被盗取的概率](http://www.cnblogs.com/forwill/p/6181984.html)
* httpOnly cookie，目前主流的浏览器基本都支持httpOnly cookie，设置httpOnly属性的cookie对javascript等脚本语言无效，从而避免了跨站攻击时js偷取cookie的情况
* 3rd-party cookie，第一方cookie是cookie种植在浏览器地址栏的域名或子域名下，第三方cookie则是种植在不同地址栏的域名下。例如，用户访问a.com时，在ad.google.com设置了cookie，在访问b.com的时候，也在ad.google.com设置了一个cookie，这种场景经常出现在google adsense、阿里妈妈之类的广告服务商中，广告商就可以采集用户的一些习惯和访问历史
* super cookie，超级cookie是设置在公共域名前缀上的cookie。例如，a.b.com的cookie可以设置在a.b.com和b.com上而不允许设置在.com上
* zombie cookie，僵尸cookie是指那些删不掉的、删掉会自动重建的cookie。僵尸cookie依赖于本地其他的存储方法，例如flash的share object、html5的local storages等，当用户删除cookie后，自动从本地其他存储中读取cookie的备份并重新种植

### 用途

1. 会话管理
* 记录用户的登录状态是cookie最常用的用途，通常web服务器会在用户登录成功后发送一个签名来标记session的有效性，免去了用户多次认证和登录
* 记录用户的访问状态，例如导航、用户的注册流程
2. 个性化信息
* cookie也经常被用来记忆用户相关的信息，以方便用户使用与自己相关的站点服务。例如，ptlogin会记忆上一次登录用户的QQ号，在下次登录时会默认填写这个QQ号
* 记忆用户自定义的一些功能，用户在设置自定义特征的时候，仅仅是保存在浏览器中，在下一次访问时，服务器会根据用户本地的cookie来展现用户的设置。例如，google将搜索设置（使用语言、每页条数以及打开搜索结果的方式等）保存在一个cookie中
3. 记录用户的行为
* 比较典型的是公司TCSS系统，它使用cookie来记录用户的点击率、某个产品或商业行为的操作率和流失率

### 实现

cookie是服务器发送给浏览器的一段文本信息，在后续的http请求中，浏览器会将cookie带回给服务器。在浏览器允许脚本执行的情况下，可以通过javascript等脚本语言设置cookie。

#### http方式

firefox浏览器，以访问`http://localhost:8080/hospital/login.jsp`为例：
* 客户端发起http请求到服务器

请求头信息
```
GET /hospital/login.jsp HTTP/1.1

Host: localhost:8080

User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64; rv:53.0) Gecko/20100101 Firefox/53.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3

Accept-Encoding: gzip, deflate

Connection: keep-alive

Upgrade-Insecure-Requests: 1
```

* 服务器返回http response，包含cookie设置

响应头信息
```
HTTP/1.1 200 OK

Server: Apache-Coyote/1.1

Set-Cookie: SESSION=0d4dd0a9-6b95-4f29-91bc-09d7c8f9bb6a; Path=/hospital/; HttpOnly

Content-Type: text/html;charset=utf-8

Content-Length: 4810

Date: Thu, 25 May 2017 07:56:10 GMT
```

* 后续访问`http://localhost:8080/hospital/`的相关页面

请求头信息
```
GET /hospital/login.jsp HTTP/1.1

Host: localhost:8080

User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64; rv:53.0) Gecko/20100101 Firefox/53.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3

Accept-Encoding: gzip, deflate

Cookie: SESSION=0d4dd0a9-6b95-4f29-91bc-09d7c8f9bb6a

Connection: keep-alive

Upgrade-Insecure-Requests: 1

Cache-Control: max-age=0
```

响应头信息
```
HTTP/1.1 200 OK

Server: Apache-Coyote/1.1

Content-Type: text/html;charset=utf-8

Content-Length: 4810

Date: Thu, 25 May 2017 08:02:55 GMT
```

#### 脚本方式

javascrip等脚本语言也可以设置cookie，[在javascript中，可以通过document.cookie实现](http://www.cnblogs.com/Jackie-sky/p/3672544.html)，例如：

```
document.cookie = "userId=828; userName=hulk";
```

### 属性

* name，cookie的名称，一旦创建名称便不可更改，cookie并不直接提供修改、删除操作，但可以通过覆盖的方式间接实现修改、删除。修改某个cookie时，只需要新建一个同名的cookie，添加到response中覆盖原来的cookie；删除某个cookie时，只需要新建一个同名的cookie并将maxAge属性设值`0`，添加到response中覆盖原来的cookie
* value，cookie的值
* domain，可以访问该cookie的域名。如果设置为`.google.com`，则所有以`google.com`结尾的域名都可以访问该cookie，其中第一个字符必须为`.`
* path，可以访问cookie的路径。如果设置path的值`/catalog`，则只有该路径及其子路径下的页面才能访问该cookie，其中最后一个字符必须为`/`
* maxAge，设置cookie的最大存活时间`单位：秒`，正值表示此数值秒后cookie将失效，负值表示cookie并非持久存储，当浏览器关闭时即删除，零值表示立即删除改cookie，默认值`–1`
* secure，cookie是否仅在使用安全协议的情况下才发送给服务器，例如，https、ssl等协议，默认值`false`
* httpOnly，对javascript等浏览器脚本语言不可见，因此跨站脚本攻击时不会被窃取
* comment，cookie的用处说明，浏览器显示cookie信息时显示该说明
* version，cookie使用的版本号，默认值`0`

### 浏览器相关

* 浏览器支持删除和禁用cookie
* 在浏览器地址栏输入`javascript:alert(document.cookie)`可以查看某个网站颁发的cookie，如下图所示
![](/image/2017/2017-05-23-java-session-cookie-1.png)
![](/image/2017/2017-05-23-java-session-cookie-2.png)

### cookie窃取及会话劫持

很多网站都使用cookie作为用户的标识，黑客可以通过窃取用户的cookie来模拟用户的请求行为，此时服务器无法辨别客户端的请求来自用户还是黑客，以下给出cookie作为用户标识的风险
* 网络监听。在网络上传输的数据都是会被监听获取，尤其是在公共的、非加密的网络环境，例如free wifi中，当黑客拿到明文的cookie之后就可以模拟用户操作，如修改密码、购物消费。解决这个问题的根本方法是使用https协议，通过ssl通道对内容以及cookie进行加密<br>
![](/image/2017/2017-05-23-java-session-cookie-3.png)

* dns cache异常。通过利用dns cache的某些漏洞，黑客可以将子域名指向自己的ip，就可以获取到包括设置了httpOnly属性的cookie，解决办法：减少dns无效配置 、isp服务商加强自我安全管理、使用https协议对请求进行加密和授权
* 跨站脚本xss窃取cookie。由于javascript等脚本语言可以读取页面文档内的cookie，也可以向任意服务器发送任意请求，因此黑客可以使用javascript等脚本语言将当前文档中的cookie据为己有。设置了httpOnly属性的cookies不能被客户端脚本读取，这降低了cookie被盗取的风险<br>
![](/image/2017/2017-05-23-java-session-cookie-4.png)

### 缺点

间谍软件利用cookie可以跟踪用户的浏览行为，更严重的是，黑客可以通过窃取cookie来获取用户的账号控制权。

### 示例

#### 步骤

* chrome浏览器，登录项目首页`http://localhost:8080/hospital/logincookie.jsp`，服务器响应头中有`Set-Cookie`属性，redis中出现新增的session信息，如下图所示
![](/image/2017/2017-05-23-java-session-cookie-5.png)
![](/image/2017/2017-05-23-java-session-cookie-6.png)

*  输入邮箱、密码，登录成功后跳转至个人中心页面，redis中并未增加新的session信息，如下图所示
![](/image/2017/2017-05-23-java-session-cookie-7.png)
![](/image/2017/2017-05-23-java-session-cookie-8.png)

* 在始终未关闭浏览器的情况下，地址栏再次输入项目首页`http://localhost:8080/hospital/logincookie.jsp`，直接跳转至个人中心页面，redis中并未增加新的session信息
* 关闭浏览器，地址栏输入项目首页`http://localhost:8080/hospital/logincookie.jsp`，直接跳转至个人中心页面，redis中出现新增的session信息，`SESSION`属性是redis中新增加的session

#### 代码

logincookie.jsp部分代码
```javascript
<script type="text/javascript">
$(function(){
	$('#email').val("");
	$('#password').val("");
	
	<%
	String emailCookie = null;
	String passwordCookie = null;
	
	Cookie[] cookies = request.getCookies(); 
	if(null != cookies){
		
		for(Cookie cookie: cookies){
			if("HOSPITAL_COOKIE_EMAIL".equals(cookie.getName())){
				emailCookie = cookie.getValue();
			}
			if("HOSPITAL_COOKIE_PASSWORD".equals(cookie.getName())){
				passwordCookie = cookie.getValue();
			}
		}
		
		if(null != emailCookie && null != passwordCookie){
			%>
			location.href = "usercentercookie.jsp";
			<%
		}
	}
	%>
});

//登录
function login(){
	var email = $('#email').val();
	var password = $('#password').val();
	
	//空值校验
	if( !email ){
		alert("邮箱不能为空，请输入");
		return false;
	}
	if( !password ){
		alert("密码不能为空，请输入");
		return false;
	}
	
	$.ajax({
		url: "api/user/login/cookie",
		async: false,
		method: "POST",
		dataType: "json",
		data: {
			email: email,
			password: password
		},
		success:function(data){
			alert(data.msg);
			if(data.code == '200'){
				location.href = "usercentercookie.jsp";
			}
		}
	});
}
</script>
```

usercentercookie.jsp部分代码
```javascript
<body>
	<h1 style="text-align: center">个人中心页面</h1>
	<div style="text-align: center">
		<p style="font-size: 15px">欢迎您，</p>
		
		<%
		String emailCookie = null;
		String passwordCookie = null;
		
		Cookie[] cookies = request.getCookies(); 
		
		if(null != cookies){
			
			for(Cookie cookie: cookies){
				if("HOSPITAL_COOKIE_EMAIL".equals(cookie.getName())){
					emailCookie = cookie.getValue();
				}
				if("HOSPITAL_COOKIE_PASSWORD".equals(cookie.getName())){
					passwordCookie = cookie.getValue();
				}
			}
			
			if(null != emailCookie && null != passwordCookie){
				%>
				<h4 style="text-align: center; font-weight: 700;margin: 20px 0;" id="email"><%=emailCookie%></h4>
				<h4 style="text-align: center; font-weight: 700;margin: 20px 0;" id="password"><%=passwordCookie%></h4>
				<%
			}
		}
		%>
	</div>
</body>
```

后端部分代码
```java
@RequestMapping(value = "/login/cookie", method = RequestMethod.POST)
@ResponseBody
public ResultInfo loginByCookie(HttpServletRequest request, HttpServletResponse response) {
	ResultInfo resultInfo = ResultInfo.newResultInfo();
	// 获取参数
	String email = request.getParameter("email");// Email
	String password = request.getParameter("password");// 登录密码

	// 空值校验
	if (StringUtils.isBlank(email) || StringUtils.isBlank(password)) {
		resultInfo.setCode(IConstant.FAILED_DATA_NOINPUT);
		resultInfo.setMsg("参数错误");
		return resultInfo;
	}

	try {
		UserBean user = new UserBean();
		user.setEmail(email);

		List<UserBean> userList = userService.getUserByParamMap(user);
		if (null != userList && userList.size() > 0) {
			UserBean userInfo = userList.get(0);
			String passwordInDB = userInfo.getPassword();
			String salt = userInfo.getPasswordsalt();

			String passwordInput = MD5Util.getSaltPassword(password, salt);
			if (null == passwordInDB) {
				resultInfo.setCode(IConstant.FAILURE);
				resultInfo.setMsg("用户信息不完整，请联系管理员");
			}
			else if (passwordInDB.equalsIgnoreCase(passwordInput)) {
				// 设置cookie
				CookieUtil.setCookie(response, IConstant.HOSPITAL_COOKIE_EMAIL, email);
				CookieUtil.setCookie(response, IConstant.HOSPITAL_COOKIE_PASSWORD, password);

				resultInfo.setCode(IConstant.SUCCESS);
				resultInfo.setMsg("登录成功");
			}
			else {
				resultInfo.setCode(IConstant.FAILURE);
				resultInfo.setMsg("密码错误");
			}
		}
		else {
			resultInfo.setCode(IConstant.FAILURE);
			resultInfo.setMsg("用户不存在");
		}
	}
	catch (Exception e) {
		log.error(e.getLocalizedMessage(), e);
		resultInfo.setCode(IConstant.FAILURE);
		resultInfo.setMsg("登录失败");
	}
	return resultInfo;
}
```

```java
/**
 * 设置cookie，指定名称及值
 *
 * @param response
 * @param name cookie的名称
 * @param value cookie的值
 */
public static void setCookie(HttpServletResponse response, String name, String value) {

	Cookie cookie = new Cookie(name, value);

	cookie.setMaxAge(IConstant.HOSPITAL_COOKIE_MAXAGE);
	cookie.setPath("/hospital/");

	response.addCookie(cookie);
}
```

## session机制

### 简介

session对象是`javax.servlet.http.HttpSession`类的实例，该对象代表一次会话（一次会话是指从客户端浏览器连接服务器开始，到客户端浏览器与服务器断开为止），当客户端浏览器与站点建立连接时，会话开始，当客户端关闭浏览器时，会话结束。session对象也是jsp脚本的一个内置对象，在jsp中可以直接使用。
<br>
<br>
session也是一种记录客户状态的机制，与cookie保存在客户端浏览器中不同，session保存在服务器上。客户端首次请求服务器时，服务器把客户信息保存在服务器，当客户端在未关闭的情况下再次请求服务器时，可从该session中查找该客户信息。
<br>
<br>
session通常用于跟踪用户的会话属性，如判断用户是否登录，或者在购物车应用中，用于跟踪用户购买的商品等。
<br>
<br>
session范围内的属性可以在多个页面间共享，一旦关闭浏览器，即session结束，session范围内的属性将全部丢失。
<br>
<br>
session机制通常用于保存客户的状态信息，这些状态信息会保存到web服务器上，所以要求session里的属性值必须是可序列化的，否则将会引发不可序列化的异常，session的属性值可以是任何可序列化的java对象。
<br>
<br>
当有多个客户访问时，服务器会保存多个客户的session，获取session的时候也不需要声明获取谁的session，session机制决定了当前客户只会获取到自己的session，而不会获取到别人的session，各客户的session彼此独立，互不可见。

### 超时

用户每访问服务器一次，无论是否读写session，服务器都认为该用户的session活跃了一次，过多的session存储在服务器中，会对服务器造成很大压力。
<br>
<br>
session的超时设置，可通过`maxInactiveInterval`属性设置单个会话的超时值，或者在web.xml文件中用`session-config`元素设置缺省超时值，也可以使用`invalidate()`方法让session直接失效，`setMaxInactiveInterval`和`session-config`方式的优先级：
* `setMaxInactiveInterval`的优先级高，如果没有使用`setMaxInactiveInterval`方法设置超时值，则默认是`session-config`中设置的时间
* `setMaxInactiveInterval`设置的是当前会话的失效时间，不是整个web服务
* `session-timeout`元素的值为零或负数时，表示会话永不超时
* `setMaxInactiveInterval`的参数单位是秒，`session-config`中`session-timeout`元素参数单位是分钟

```xml
<session-config>
	<session-timeout>30</session-timeout>
</session-config>
```

* 使用`invalidate()`方法能直接使session失效

### 方法

* setAttribute(String name, Object value)，将对象绑定至session并指定`name`，`name`参数不能为空，已有同`name`的session时，替换值，`value`参数为空时，作用与removeAttribute(String name)方法相同
* getAttribute(String name)，从session中获得绑定至`name`的值，当没有值被绑定至此`name`时，返回`null`
* removeAttribute(String name)，从session中移除绑定至`name`的对象，当没有值被绑定至此`name`时，方法等同于未执行
* getAttributeNames()，获取与此session关联的所有`name`
* invalidate()，使session失效，解除绑定至此session的所有对象
* getId()，获取session的id，id由servlet容器分配
* getCreationTime()，session创建的时间，1/1/1970 GMT以来的毫秒
* getLastAccessedTime，获取与此session关联的客户端最后一次请求的时间，1/1/1970 GMT以来的毫秒，并作为容器收到请求的时间
* setMaxInactiveInterval(int interval)，session的超时时间，servlet容器使session失效前客户端请求之间的最大间隔时间，单位是秒
* getMaxInactiveInterval()，返回servlet容器使session处于活跃状态的客户端请求间的最大间隔时间，单位是秒
* isNew()，假如客户端并不知道此session或客户端禁用session时，返回`true`，例如，假如服务器只使用基于cookie的session，而客户端禁用cookie时，每个请求上的session将是新创建的

```java
HttpSession session = request.getSession();

// user是可序列化的对象
session.setAttribute("_USER_", user);
Object userValue = session.getAttribute("_USER_");

// session id
String sessionId = session.getId();
// create time
long createTime = session.getCreationTime();
// session的最后活跃时间
long lastAccessedTime = session.getLastAccessedTime();
// session是否新创建
session.isNew();

// session的超时时间
session.setMaxInactiveInterval(10 * 60);
session.getMaxInactiveInterval();

// names
Enumeration er = session.getAttributeNames();
while (er.hasMoreElements()) {
	Object obj = er.nextElement();
	System.out.println(obj);
}

// 移除绑定至session的name
session.removeAttribute("_USER_");
// 使session失效
session.invalidate();
```

### 浏览器

虽然session保存在服务器上，对客户端浏览器透明，但是它的正常运行仍需要客户端浏览器的支持，http协议是无状态的协议，session不能依据http连接来判断客户端是否为同一用户，session需要使用cookie作为识别标志。
<br>
<br>
服务器会向客户端浏览器发送一个名为`SESSION`的cookie，它的值是该session的id，即`getId()`方法的返回值，session依据该cookie来识别是否为同一用户。该cookie为服务器自动生成，它的`maxAge`属性一般为`–1`，仅在当前浏览器内有效，且各浏览器窗口间不共享，关闭浏览器即失效。session方式，登录前、后的firefox浏览器cookie变化情况如下图所示：<br>
![](/image/2017/2017-05-23-java-session-cookie-9.png)
![](/image/2017/2017-05-23-java-session-cookie-10.png)

### URL重写

URL重写是客户端浏览器不支持cookie时的一个解决方案，原理是将用户的session id信息重写到URL中，服务器通过解析重写后的URL以获取session id，即使客户端不支持cookie，仍可以使用session来记录用户状态。`HttpServletResponse`类提供了`encodeURL(String url)`方法实现URL重写。

* [本文所用的完整项目代码托管在github](https://github.com/soyuone/hospital)
* [参考：《疯狂java讲义》]()
* [参考：cookie和session有什么区别](https://www.zhihu.com/question/19786827)
* [参考：cookie、session机制详解](http://blog.csdn.net/fangaoxin/article/details/6952954/)
* [参考：http cookie详解](http://blog.csdn.net/lijing198997/article/details/22174151)
* [参考：cookie和session详解](http://www.cnblogs.com/shiyangxt/archive/2008/10/07/1305506.html)