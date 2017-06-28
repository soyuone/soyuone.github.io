---
layout: post
title: "java异常处理机制"
categories: java
tags: java exception
author: Kopite
---

* content
{:toc}


java的异常处理机制将业务代码和错误处理分离，让程序有更好的容错性，使程序更加健壮。java把所有的非正常情况分为两种：exception、error，它们都继承自throwable父类。
<br>
<br>
error通常是与虚拟机相关的问题，如系统崩溃、虚拟机错误、动态链接失败等，[这种错误无法恢复或不可能捕获](http://blog.csdn.net/androidyue/article/details/9531293)，将导致程序中断，程序不应该试图使用catch块来捕获error。
<br>
<br>
exception分为两种：`checked异常`、`runtime异常`，所有的runtimeexception类及其子类的实例被称为`runtime异常`，不是runtimeexception类及其子类的实例则被称为`checked异常`。`checked异常`是可以在编译阶段被处理的异常，所以java强制程序处理所有的checked异常，否则无法通过编译，而`runtime异常`在编译阶段则无须处理。
<br>
<br>
java的异常处理机制主要依赖于try、catch、finally、throw及throws五个关键字，try块中存放可能引发异常的代码，catch块用于捕获并处理指定类型及其子类的异常，finally块用于回收在try块中打开的物理资源，throws主要在方法签名中使用，用于声明该方法可能抛出的异常，throw用于在代码中抛出一个具体的异常。



## 异常处理机制

* try块中的业务代码出现异常时，系统自动生成一个异常对象，该异常对象被提交给java运行时环境，此过程称为throw异常，当java运行时环境收到异常对象时会寻找能处理该异常对象的catch块，此过程称为catch异常。通常情况下，try块后只会有一个catch块会被执行，java异常类的继承关系如下两图所示：
![](/image/2017/2017-05-14-java-exception-1.jpg)
![](/image/2017/2017-05-14-java-exception-2.jpeg)
<br>
* 进行异常捕获时要将父类异常的catch块放在子类异常catch块的后面，否则将出现：
```
Unreachable catch block for *. It is already handled by the catch block for *
```
* throws声明抛出异常只能在方法签名中使用，可以声明抛出多个异常类，多个异常类之间用逗号隔开，一旦使用throws声明抛出异常，程序就无须使用try...catch来捕获异常，该异常将交给上一级调用者处理
* java的垃圾回收机制不会回收任何物理资源，gc只能回收堆内存中对象所占用的内存，程序在try块中打开的物理资源通常要显式回收，为此java提供了finally块。不管try块中的代码是否出现异常，也不管哪一个catch块被执行，甚至在try块或catch块中执行了return语句，finally块总会被执行，只有当finally块执行完成后，系统才会再次跳回来执行try块、catch块里的return或throw语句
* 除非在try块、catch块中调用了退出虚拟机的方法，否则不论在try块、catch块中执行怎样的代码，异常处理的finally块总会被执行。通常情况下，不要在finally块中使用如return或throw等导致方法终止的语句，一旦在finally块中使用了return或throw语句，将会导致try块、catch块中的return、throw语句失效，系统将不会跳回去执行try块、catch块里的代码
* 异常处理结构中只有try块是必须，catch块和finally块都是可选但至少二者出现其一，catch块必须位于try块之后，finally块必须位于所有的catch块之后

TestException.java
```java
public class TestException {

	/**
	 * 构造器
	 */
	public TestException() {

	}

	/**
	 * testEx方法
	 */
	boolean testEx() throws Exception {
		boolean ret = true;
		try {
			// 调用testEx1方法
			ret = testEx1();
		}
		catch (Exception e) {
			System.out.println("testEx, catch exception");
			ret = false;
			throw e;
		}
		finally {
			System.out.println("testEx, finally; return value=" + ret);
			return ret;
		}
	}

	/**
	 * testEx1方法
	 */
	boolean testEx1() throws Exception {
		boolean ret = true;
		try {
			// 调用testEx2方法
			ret = testEx2();
			if (!ret) {
				return false;
			}
			System.out.println("testEx1, at the end of try");
			return ret;
		}
		catch (Exception e) {
			System.out.println("testEx1, catch exception");
			ret = false;
			throw e;
		}
		finally {
			System.out.println("testEx1, finally; return value=" + ret);
			return ret;
		}
	}

	/**
	 * testEx2方法
	 */
	boolean testEx2() throws Exception {
		boolean ret = true;
		try {
			int b = 12;
			int c;
			for (int i = 2; i >= -2; i--) {
				c = b / i;
				System.out.println("i=" + i);
			}
			return true;
		}
		catch (Exception e) {
			System.out.println("testEx2, catch exception");
			ret = false;
			throw e;
		}
		finally {
			System.out.println("testEx2, finally; return value=" + ret);
			return ret;
		}
	}

	/**
	 * main方法
	 */
	public static void main(String[] args) {
		TestException testException1 = new TestException();
		try {
			// 调用testEx方法
			testException1.testEx();
		}
		catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
<br>
控制台输出：
```
i=2
i=1
testEx2, catch exception
testEx2, finally; return value=false
testEx1, finally; return value=false
testEx, finally; return value=false
```

## 处理checked异常

* 当前方法明确知道如何处理该异常时，使用try...catch来捕获并处理
* 当前方法不知道如何处理该异常时，在定义该方法时throws声明抛出。需要注意的是当使用throws抛出异常时，子类方法声明抛出的异常类型应该是父类方法声明抛出异常的子类或相同，并且不允许子类声明抛出的异常比父类声明抛出的异常多

## 处理runtime异常

* runtime异常无须显式声明抛出，如果需要捕获runtime异常，也可以使用try...catch块来实现

TestController.java
```java
	/**
	 * <p>
	 * Description:[exception测试]
	 * </p>
	 * Created by [SOYU] [2017年5月14日] Midified by [修改人] [修改时间]
	 *
	 * @return
	 */
	@RequestMapping(value = "/exception", method = RequestMethod.GET)
	@ResponseBody
	public ResultInfo exceptionThrow() {
		ResultInfo resultInfo = new ResultInfo();
		try {
			userService.exceptionThrow();
			
			resultInfo.setCode(IConstant.SUCCESS);
			resultInfo.setMsg("成功");
		}
		catch (Exception e) {
			resultInfo.setCode(IConstant.FAILURE);
			resultInfo.setMsg("失败");
		}
		return resultInfo;
	}
```
<br>
UserService.java
```java
public void exceptionThrow();
```
<br>
UserServiceImpl.java
```java
	@SuppressWarnings("unused")
	@Override
	public void exceptionThrow() {
		int a = 2/0;
	}
```
<br>
返回结果：
```json
{
"code": 500,
"msg": "失败"
}
```

## 异常链

实际应用常采用分层结构，程序先捕获原始异常，然后抛出一个新的业务异常，新的业务异常中包含了对用户的提示信息，这种方式被称为`异常转译`。通过`异常转译`可以把原始异常信息隐藏起来，仅向上提供必要的异常提示信息，使底层异常不会扩散到表现层，避免暴露太多的实现细节。捕获一个异常然后接着抛出另一个异常并把原始异常信息保存下来是一种典型的链式处理，因此也被称为`异常链`。

* [参考：《疯狂java讲义》]()
* [参考：深入理解java异常处理机制](http://blog.csdn.net/hguisu/article/details/6155636/)
* [参考：java异常类层次结构](http://blog.csdn.net/renfufei/article/details/16344847)