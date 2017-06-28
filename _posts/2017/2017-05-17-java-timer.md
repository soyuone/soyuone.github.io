---
layout: post
title: "java定时任务"
categories: java
tags: java 定时器
author: Kopite
---

* content
{:toc}


项目中有用java做定时任务，此处翻译`spring framework reference documentation v4.2.0`中`Part VII. Integration - 33. Task Execution and Scheduling`部分，并小结定时任务的几种不同实现方式。



## 33. Task Execution and Scheduling

### 33.1 简介

`spring framework`分别使用`TaskExecutor`、`TaskScheduler`接口对异步执行(asynchronous execution)和任务调度(scheduling of tasks)进行了抽象。`spring`也提供了对支持线程池接口的实现，并且也可以在应用服务器环境下对CommonJ进行委派。`spring`对这些常用接口的实现基本上可以消除Java SE 5，Java SE 6和Java EE环境之间的差异。
<br>
<br>
`spring`对Timer定时器和[Quartz定时器](http://quartz-scheduler.org)进行了整合，其中Timer是JDK 1.3以来自带的定时器。这两个定时器都使用FactoryBean进行设置，分别引用Timer、Trigger实例。此外，`spring`提供了一个用于Quartz Scheduler和Timer的类，可以调用现有目标对象的方法(类似于普通MethodInvokingFactoryBean操作)。

### 33.2 TaskExecutor抽象



### 33.3 TaskScheduler抽象

除了对`TaskExecutor`的抽象，`spring 3.0`还引入了有多种方法的`TaskScheduler`，实现在将来某个时间点运行任务。
```java
public interface TaskScheduler {

	ScheduledFuture schedule(Runnable task, Trigger trigger);

	ScheduledFuture schedule(Runnable task, Date startTime);

	ScheduledFuture scheduleAtFixedRate(Runnable task, Date startTime, long period);

	ScheduledFuture scheduleAtFixedRate(Runnable task, long period);

	ScheduledFuture scheduleWithFixedDelay(Runnable task, Date startTime, long delay);

	ScheduledFuture scheduleWithFixedDelay(Runnable task, long delay);
}
```
上述接口中最简单的一个方法是`schedule(Runnable task, Date startTime)`，该方法在指定时间调度任务执行一次，除此之外的其他所有方法都能调度任务重复执行。虽然固定速率、固定延迟的方法相对简单、周期性执行，但使用触发器的方法更加灵活。

#### Trigger接口

`Trigger`接口是受JSR-236的启发，JSR-236在`spring 3.0`中还没有被实现。触发器的基本思想是执行时间可以基于过去的执行结果甚至任意条件。假如这些决定把上一次执行的结果考虑在内，则上述思想在`TriggerContext`中可以实现，`Trigger`接口如下所示。
```java
public interface Trigger {

 Date nextExecutionTime(TriggerContext triggerContext);
}
```
如上所示，`TriggerContext`是`Trigger`接口的最重要部分，它封装了全部相关数据，并且在未来有需要时可以扩展，`TriggerContext`接口如下所示。
```java
public interface TriggerContext {

	Date lastScheduledExecutionTime();

	Date lastActualExecutionTime();

	Date lastCompletionTime();
}
```

#### Trigger实现类

`spring`为`Trigger`接口提供了两个实现类，`CronTrigger`类是其中一个实现类，基于cron表达式的定时器成为可能，以下示例定时任务设置在星期一至星期五上午09:00-17:00间每小时的15分运行。
```java
scheduler.schedule(task, new CronTrigger("* 15 9-17 * * MON-FRI"));
```
另一个开箱即用的实现是`PeriodicTrigger`，它可以接受一个固定的周期、一个可选的初始延迟值和一个指示该周期是否该被解读为固定速率或固定延迟的布尔值。`TaskScheduler`接口定义方法执行定时任务在一个固定速率或一个固定延迟，可以在任意时刻运行。`PeriodicTrigger`实现类的价值在于它可以在依赖于Trigger抽象类的组件中使用。例如，它允许周期性触发器、基于cron表达式的触发器甚至自定义触发器的实现类可以互换使用。这类组件可以利用依赖注入的优点，以便可以在外部配置这些触发器。

#### TaskScheduler实现类

与`spring`的`TaskExecutor`抽象类一样，`TaskScheduler`的主要优点是依赖于调度行为的代码不需要耦合到特定定时器的实现。当在Application Server环境中运行时，它提供的灵活性尤其重要，因为应用程序本身不直接创建线程。对于这些情况，`spring`提供了`TimerManagerTaskScheduler`，用于委托给CommonJ TimerManager实例，通常配置有JNDI-lookup。
<br>
<br>
一个更简单的选择，`ThreadPoolTask​​Scheduler`可以在外部线程管理不是必需的时候使用。在内部，它委托给`ScheduledExecutorService`实例。`ThreadPoolTask​​Scheduler`实际上也实现了`spring`的`TaskExecutor`接口，以使单个实例同调度一样能尽快的被用于异步执行，并且可以重复执行。

### 33.4 基于注解的定时及异步执行

`spring`为任务调度及异步方法执行提供了注解支持。

#### 启用scheduling注解

为使用`@Scheduled`、`@Async`，在一个`@Configuration`类中添加`@EnableScheduling`及`@EnableAsync`。
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.EnableScheduling;

@Configuration
@EnableAsync
@EnableScheduling
public class AppConfig {
}
```
可以自由的为应用程序选择使用相关注解，例如，只需要`@Scheduled`支持时可以省去`@EnableAsync`。对于更细粒度的控制，可以另外实现`SchedulingConfigurer`和/或`AsyncConfigurer`接口，查看javadocs以获得更详细信息。
<br>
<br>
使用XML文件进行配置时，使用`<task:annotation-driven>`元素。
```xml
<task:annotation-driven executor="myExecutor" scheduler="myScheduler"/>
<task:executor id="myExecutor" pool-size="5"/>
<task:scheduler id="myScheduler" pool-size="10"/>
```
上述XML文件提供了一个executor引用来处理与`@Async`注解的相关方法，提供了scheduler引用来管理@Scheduled注解的方法。

#### @Scheduled注解

`@Scheduled`注解可以同trigger metadata一起添加到方法中。例如，以下方法将以固定的延迟每5秒被调用，这意味着从每次前面调用完成时开始测量周期。
```java
@Scheduled(fixedDelay = 5000)
public void doSomething() {
	// something that should execute periodically
}
```
如果需要以固定速率执行，只需更改注解中指定的属性名称。以下操作每5秒被执行，在每次调用的开始时被计时。
```java
@Scheduled(fixedRate = 5000)
public void doSomething() {
	// something that should execute periodically
}
```
对于固定延迟、固定速率的任务，可以指定初始延迟，延迟指示了在该方法第一次执行之前等待的毫秒数。
```java
@Scheduled(initialDelay = 1000, fixedRate = 5000)
public void doSomething() {
	// something that should execute periodically
}
```
简单的周期性调度不满足使用，提供了cron表达式，以下示例任务仅在周一至周五执行。
```java
@Scheduled(cron = "*/5 * * * * MON-FRI")
public void doSomething() {
	// something that should execute on weekdays only
}
```
使用zone属性可以在cron表达式中指定时区。值得注意的是，定时方法返回值必须为void、并且不能传入任何参数。如果该方法需要与应用程序上下文中的其他对象进行交互，则通常将通过依赖注入来提供这些对象。
<br>
<br>
在运行时，确保不初始化多个用`@Scheduled`注解的实例，除非想要调度每个这样的实例的回调。与此相关，确保不使用`@Configurable`注解那些用`@Scheduled`注解、使用容器注入为常规`spring`Bean的Bean：否则将获得双重初始化——每个`@Scheduled`注解的方法被调用两次的结果，一次通过容器注入初始化，一次通过`@Configurable`初始化。

#### @Async注解

用`@Async`注解方法能让该方异步执行，换句话说，调用者将在调用后立即返回，并且该方法的实际执行将发生在已提交给`spring``TaskExecutor`的任务中。最简单的情况，`@Async`注解于void返回方法。
```java
@Async
void doSomething() {
	// this will be executed asynchronously
}
```
与用`@Scheduled`注解的方法不同，`@Async`注解的方法方法可以接收参数，因为调用者在运行时以“正常”方式调用它们，而不是由容器管理的调度任务调用。以下示例是可以正常运行
```java
@Async
void doSomething(String s) {
	// this will be executed asynchronously
}
```
即使有返回值的方法也可以异步调用，此时需要有`Future`返回类型的值。这仍然显示了异步执行的好处，调用者可以在该`Future`上调用get（）之前执行其他任务。
```java
@Async
Future<String> returnSomething(int i) {
	// this will be executed asynchronously
}
```
`@Async`不能与lifecycle callbacks（例如`@PostConstruct`）结合使用。为了异步初始化`spring` bean，当前必须使用一个单独的初始化`spring` bean，然后由它调用`@Async`注解的目标方法。
```java
public class SampleBeanImpl implements SampleBean {
	@Async
	void doSomething() {
		// ...
	}
}

public class SampleBeanInititalizer {
	private final SampleBean bean;

	public SampleBeanInitializer(SampleBean bean) {
		this.bean = bean;
	}

	@PostConstruct
	public void initialize() {
		bean.doSomething();
	}
}
```

#### @Async中限定Executor

默认情况下，当使用`@Async`注解方法时，使用的执行器是`annotation-driven`元素中描述的。当使用指定执行器时，可以使用`@Async`的`value`属性。
```java
@Async("otherExecutor")
void doSomething(String s) {
	// this will be executed asynchronously by "otherExecutor"
}
```
上述示例中，"otherExecutor"可以是`spring`容器中任意`Executor` bean，也可以是与任何Executor相关联的qualifier的名称，举例来说，<qualifier>元素限定或`spring``@Qualifier`注解。

#### @Async管理异常

当`@Async`注解的方法有`Future`类型返回值时，在方法执行期间管理异常很容易，因为在调用`Future`中`get`时异常会被抛出。返回结果为void时，异常不可捕获并且不可传递，此时可以抛出`AsyncUncaughtExceptionHandler`异常。
```java
public class MyAsyncUncaughtExceptionHandler implements AsyncUncaughtExceptionHandler {
	@Override
	public void handleUncaughtException(Throwable ex, Method method, Object... params) {
		// handle exception
	}
}
```
默认情况下，异常会被记录。使用`AsyncConfigurer`或`task:annotation-driven`XML元素可以自定义AsyncUncaughtExceptionHandler。

### 33.5 Task命名空间



### 33.6 Quartz



## spring task定时器示例

applicationContext.xml
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

	<!-- 属性占位符配置器-容器后处理器：在容器实例化其他Bean之前会自动读取配置文件，ApplicationContext会自动检测在容器中的容器后处理器并自动注册它们 -->
	<!-- 多个属性占位符配置器，http://blog.csdn.net/sunhuwh/article/details/15813103 -->
	<bean id="propertyConfigurer"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:data/druid.properties</value>
				<value>classpath:data/redis.properties</value>
				<value>classpath:data/thread.properties</value>
			</list>
		</property>
	</bean>

	<!-- 自动扫描指定包及其子包下所有符合条件的Bean，排除@Controller标注的Bean，扫描@Component、@Service、@Repository标注的Bean -->
	<context:component-scan base-package="com.song.hospital">
		<context:exclude-filter type="annotation"
			expression="org.springframework.stereotype.Controller" />
	</context:component-scan>

	<!-- spring task定时器开关 -->
	<task:annotation-driven />

	<!-- 加载classes中的指定文件 -->
	<import resource="classpath:mybatis/spring-datasource-druid.xml" />
	<import resource="classpath:spring/spring-redis.xml" />
	<import resource="classpath:spring/spring-session.xml" />
	<import resource="classpath:spring/spring-thread.xml" />

	<!-- NoSuchBeanDefinitionException: No qualifying bean of type [org.springframework.scheduling.TaskScheduler] 
		is defined，http://blog.csdn.net/oarsman/article/details/52801877 -->
</beans>
```
<br>
SpringTaskService.java
```java
package com.song.hospital.service;

/**
 * <p>
 * Title: hospital_[子系统名称]_[spring task]
 * </p>
 * <p>
 * Description: [spring task Service]
 * </p>
 * 
 * @author SOYU
 * @version $Revision$ 2017年5月19日
 * @author (lastest modification by $Author$)
 * @since 20100901
 */
public interface SpringTaskService {

	/**
	 * <p>
	 * Description:[定时任务，每天早上09:00触发，查询用户数量]
	 * </p>
	 * Created by [SOYU] [2017年5月19日] Midified by [修改人] [修改时间]
	 */
	public void springTaskAtMorning();

	/**
	 * <p>
	 * Description:[定时任务，每周一09:15触发，查询用户数量]
	 * </p>
	 * Created by [SOYU] [2017年5月19日] Midified by [修改人] [修改时间]
	 */
	public void springTaskAtMonday();

	/**
	 * <p>
	 * Description:[定时任务，每月一号09:30触发，查询用户数量]
	 * </p>
	 * Created by [SOYU] [2017年5月19日] Midified by [修改人] [修改时间]
	 */
	public void springTaskAtMonth();

}

```
<br>
SpringTaskServiceImpl.java
```java
package com.song.hospital.service.impl;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

import com.song.hospital.service.SpringTaskService;
import com.song.hospital.service.UserService;

/**
 * <p>
 * Title: hospital_[子系统名称]_[spring task]
 * </p>
 * <p>
 * Description: [spring task ServiceImpl]
 * </p>
 * 
 * @author SOYU
 * @version $Revision$ 2017年5月19日
 * @author (lastest modification by $Author$)
 * @since 20100901
 */
@Service("springTaskServiceImpl")
public class SpringTaskServiceImpl implements SpringTaskService {

	private Logger log = LoggerFactory.getLogger(SpringTaskServiceImpl.class);

	@Autowired
	private UserService userService;

	@Scheduled(cron = "0 0 9 * * ?")
	@Override
	public void springTaskAtMorning() {
		try {
			Integer userNum = userService.countBy(null);
			System.out.println("截止到今天09:00的用户数量为：" + userNum);
		}
		catch (Exception e) {
			log.error(e.getLocalizedMessage(), e);
		}
	}

	@Scheduled(cron = "0 15 9 ? * MON")
	@Override
	public void springTaskAtMonday() {
		try {
			Integer userNum = userService.countBy(null);
			System.out.println("截止到今天09:15的用户数量为：" + userNum);
		}
		catch (Exception e) {
			log.error(e.getLocalizedMessage(), e);
		}
	}

	@Scheduled(cron = "0 30 9 1 * ?")
	@Override
	public void springTaskAtMonth() {
		try {
			Integer userNum = userService.countBy(null);
			System.out.println("截止到今天09:30的用户数量为：" + userNum);
		}
		catch (Exception e) {
			log.error(e.getLocalizedMessage(), e);
		}
	}

}
```

* [本文所用的完整项目代码托管在github](https://github.com/soyuone/hospital)
* [参考：spring framework reference documentation](http://spring.io/docs/reference)
* [参考：深入浅出spring task定时任务](http://blog.csdn.net/u011116672/article/details/52517247)
