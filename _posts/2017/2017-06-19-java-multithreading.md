---
layout: post
title: "java基础：多线程"
categories: java
tags: java 多线程
author: Kopite
---

* content
{:toc}


单线程的程序只有一个顺序执行流，多线程的程序则可以包括多个顺序执行流，多个顺序流之间互补干扰。可以这样理解：单线程的程序如同雇佣一个服务员的餐厅，他必须做完一件事情后才可以做下一件事情；多线程的程序如同雇佣多个服务员的餐厅，他们可以同时做多件事情。
<br>
<br>
java语言提供了非常优秀的多线程支持，程序可以通过简单的方式来启动多线程。本文将会详细介绍java多线程编程的相关方面，包括创建、启动线程、控制线程，以及多线程的同步操作，并会介绍如何利用java内建支持的`线程池`来提高多线程性能。



## 概述

几乎所有的操作系统都支持同时运行多个任务，一个任务通常就是一个程序，每个运行中的程序就是一个进程。当一个程序运行时，内部可能包含了多个顺序执行流，每个顺序执行流就是一个线程。

### 线程和进程

当一个`程序`进入内存运行时，即变成一个`进程`。`进程`是处于运行过程中的程序，并且具有一定的独立功能，`进程是系统进行资源分配和调度的一个独立单位`。一般而言，进程包含如下三个特征：
* 独立性：进程是系统中独立存在的实体，它可以拥有自己独立的资源，每一个进程都拥有自己私有的地址空间。在没有经过进程本身允许的情况下，一个用户进程不可以直接访问其它进程的地址空间
* 动态性：`进程`和`程序`的区别在于，程序只是一个静态的指令集合，而进程是一个正在系统中活动的指令集合。在进程中加入了时间的概念。进程具有自己的生命周期和各种不同的状态，这些概念在程序中都是不具备的
* 并发性：多个进程可以在单个处理器上`并发`执行，多个进程之间不会互相影响

注意：`并行性`是指在同一时刻，有多条指令在多个处理器上同时执行。`并发性`是指在同一时刻只能有一条指令执行，但多个进程指令被快速轮换执行，使得在宏观上具有多个进程同时执行的效果。
<br>
<br>
大部分操作系统都支持多进程并发执行，现代的操作系统几乎都支持同时运行多个任务。例如，边敲代码边听音乐...，这些进程开上去像是在同时工作。但事实的真相是，对于一个CPU而言，它在某个时间点只能执行一个程序，也就是说，只能运行一个进程，CPU不断的在这些进程之间轮换执行。
<br>
<br>
目前操作系统大多采用效率较高的`抢占式多任务操作策略`，例如UNIX/Linux等。
<br>
<br>
`多线程`扩展了多进程的概念，使得同一个进程可以同时并发处理多个任务。`线程（Thread）`也被称作`轻量级进程`，`线程是进程的执行单元`。就像进程在操作系统中的地位一样，`线程在程序中是独立的、并发的执行流`。当进程被初始化后，`主线程`就被创建了。对于绝大多数的应用程序来说，通常仅要求有一个主线程，但也可以在该进程内创建多条顺序执行流，这些顺序执行流就是线程，每个线程互相独立。
<br>
<br>
`线程是进程的组成部分，一个进程可以拥有多个线程，一个线程必须有一个父进程`。`线程可以拥有自己的堆栈、自己的程序计数器和自己的局部变量，但不拥有系统资源，它与父进程的其它线程共享该进程所拥有的全部资源`。`线程可以完成一定的任务，可以与其它线程共享父进程中的共享变量及部分环境，相互之间协同来完成进程所要完成的任务`。
<br>
<br>
线程是独立运行的，它并不知道进程中是否还有其他线程存在。`线程的执行是抢占式的，当前运行的线程在任何时候都可能被挂起，以便另外一个线程可以运行`。
<br>
<br>
`一个线程可以创建和撤销另一个线程，同一个进程中的多个线程之间可以并发执行`。
<br>
<br>
`线程的调度和管理由进程本身负责完成`。简而言之，`一个程序运行后至少有一个进程，一个进程里可以包含多个线程，但至少要包含一个线程`。`操作系统可以同时执行多个任务，每个任务就是进程，进程可以同时执行多个任务，每个任务就是线程`。

### 多线程的优势

线程在程序中是独立的、并发的执行流，与分隔的进程相比，进程中线程之间的隔离程度要小。它们`共享内存、文件句柄和其他每个进程应有的状态`。
<br>
<br>
因为线程的划分尺度小于进程，使得多线程程序的并发性高。`进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大的提高了程序的运行效率`。
<br>
<br>
线程比进程具有更高的性能，这是由于同一个进程中的线程都有共性——多个线程共享同一个进程虚拟空间。`线程共享的环境包括：进程代码段、进程的公有数据等`。利用这些共享的数据，线程很容易实现相互之间的通信。
<br>
<br>
当操作系统创建一个进程时，必须为该进程分配独立的内存空间，并分配大量的相关资源，但创建一个线程则简单的多，因此`使用多线程来实现并发比使用多进程实现并发的性能要高的多`，多线程编程的优点总结如下：
* 进程间不能共享内存，但线程间共享内存非常容易
* 系统创建进程时需要为该进程重新分配系统资源，但创建线程则代价小的多，因此使用多线程来实现多任务并发比多进程的效率高
* java语言内置了多线程功能支持，而不是单纯的作为底层操作系统的调度方式，从而简化了java的多线程编程

## 创建线程

java使用`java.lang.Thread`类代表线程，所有的线程对象都必须是`java.lang.Thread`或其子类的实例。每个线程的作用是完成一定的任务，实际上就是执行一段顺序执行的代码。java使用线程执行体来代表这段程序流。

### Thread类

通过继承`java.lang.Thread`类创建并启动多线程的步骤如下：
* 定义`java.lang.Thread`类的子类，并重写`run()`方法，`run()`方法的方法体就代表了线程需要完成的任务，因此把`run()`方法称为`线程执行体`
* 创建`java.lang.Thread`子类的实例，即创建了线程对象
* 调用线程对象的`start()`方法来启动该线程

```java
public class FirstThread extends Thread {

	private int i;

	// 重写run方法，run()方法的方法体就是线程执行体
	@Override
	public void run() {
		for (; i < 20; i++) {
			// 当线程类继承Thread类时，直接使用this即可获取当前线程
			// Thread对象的getName()方法返回当前线程的名字
			System.out.println("run：" + getName() + " " + i);
		}
	}

	public static void main(String[] args) throws InterruptedException {

		// 为主线程设置名称，默认名称为main
		Thread.currentThread().setName("main：");

		for (int i = 0; i < 20; i++) {
			// 调用Thread类的currentThread()方法获取当前线程
			System.out.println(Thread.currentThread().getName() + " " + i);

			if (i == 10) {
				// 创建并启动第一个线程
				FirstThread firstThread = new FirstThread();
				firstThread.setName("first：");
				firstThread.start();

				// 创建并启动第二个线程
				FirstThread secondThread = new FirstThread();
				secondThread.setName("second：");
				secondThread.start();
			}
		}
	}

}
```

* `Thread java.lang.Thread.currentThread()`是Thread类的静态方法，该方法总是返回当前正在执行的线程对象
* `void java.lang.Thread.setName(String name)`是Thread类的实例方法，用于设置线程的名称
* `String java.lang.Thread.getName()`是Thread类的实例方法，该方法返回调用该方法的线程名称

上述示例的main()方法中包含了一个循环，当循环变量i == 10时创建并启动两个新线程，运行结果如下所示：

```
main： 0
main： 1
main： 2
main： 3
main： 4
main： 5
main： 6
main： 7
main： 8
main： 9
main： 10
main： 11
main： 12
main： 13
main： 14
main： 15
main： 16
main： 17
main： 18
main： 19
run：first： 0
run：first： 1
run：first： 2
run：first： 3
run：first： 4
run：first： 5
run：first： 6
run：first： 7
run：first： 8
run：first： 9
run：second： 0
run：first： 10
run：first： 11
run：first： 12
run：first： 13
run：first： 14
run：first： 15
run：first： 16
run：first： 17
run：first： 18
run：first： 19
run：second： 1
run：second： 2
run：second： 3
run：second： 4
run：second： 5
run：second： 6
run：second： 7
run：second： 8
run：second： 9
run：second： 10
run：second： 11
run：second： 12
run：second： 13
run：second： 14
run：second： 15
run：second： 16
run：second： 17
run：second： 18
run：second： 19
```

上述示例中虽然只显式的创建了2个线程，但实际上程序有3个线程，即程序显式创建的2个子线程和主线程。`当java程序开始运行后，程序至少会创建一个主线程，主线程的线程执行体不是由run()方法确定的，而是由main()方法确定——main()方法的方法体代表主线程的线程执行体`。
<br>
<br>
另外，`first：`线程和`second：`线程输出的 i 变量不连续—— i 变量是`FirstThread`类的实例变量。因为程序每次创建线程对象时都需要创建一个`FirstThread`对象，所以`first：`线程和`second：`线程不能共享该实例变量。通过继承`java.lang.Thread`类创建线程时，多个线程间无法共享线程类的实例变量。

### Runnable接口

通过实现`java.lang.Runnable`接口创建并启动多线程的步骤如下：
* 定义`java.lang.Runnable`接口的实现类，并重写该接口的`run()`方法，该`run()`方法的方法体即是该线程的`线程执行体`
* 创建`java.lang.Runnable`实现类的实例，并以此实例作为`java.lang.Thread`类的`target`来创建Thread对象，该Thread对象才是真正的线程对象
```java
java.lang.Thread.Thread(Runnable target, String name)
```
* 调用线程对象的`start()`方法来启动该线程

```java
public class SecondThread implements Runnable {

	private int i;

	// 重写run方法，run()方法同样是线程执行体
	@Override
	public void run() {
		for (; i < 20; i++) {
			// 当线程类实现Runable接口时，如果想获取当前线程，只能用Thread.currentThread()方法
			// Thread对象的getName()方法返回当前线程的名称
			System.out.println("run：" + Thread.currentThread().getName() + " " + i);
		}
	}

	public static void main(String[] args) {

		// 为主线程设置名称，默认名称为main
		Thread.currentThread().setName("main：");

		for (int i = 0; i < 20; i++) {
			// 调用Thread类的currentThread()方法获取当前线程
			System.out.println(Thread.currentThread().getName() + " " + i);

			if (i == 10) {
				SecondThread st = new SecondThread();

				// 通过new Thread(target, name)方法创建新线程
				// 创建并启动第一个线程
				Thread firstThread = new Thread(st, "first：");
				firstThread.start();

				// 创建并启动第二个线程
				Thread secondThread = new Thread(st, "second：");
				secondThread.start();
			}
		}
	}

}
```

* `java.lang.Thread.Thread(Runnable target, String name)`，可在创建Thread对象时为其指定名称

Runnable对象仅仅作为Thread对象的target，`java.lang.Runnable`实现类里包含的`run()`方法仅作为`线程执行体`。而实际的线程对象依然是`java.lang.Thread`实例，只是该Thread线程负责执行其target的run()方法。
<br>
<br>
用继承`java.lang.Thread`类的方式创建线程，获得当前线程对象直接使用this即可。通过实现`java.lang.Runnable`接口方式获得当前线程对象，必须使用`Thread.currentThread()`方法。并且前者创建的Thread子类即可代表线程对象，而后者创建的Runnable对象只能作为线程对象的target。
<br>
<br>
上述示例的main()方法中包含了一个循环，当循环变量i == 10时创建并启动两个新线程，运行结果如下所示：

```
main： 0
main： 1
main： 2
main： 3
main： 4
main： 5
main： 6
main： 7
main： 8
main： 9
main： 10
main： 11
run：first： 0
main： 12
run：second： 0
main： 13
run：first： 1
main： 14
run：second： 2
main： 15
run：first： 3
main： 16
run：second： 4
main： 17
run：first： 5
main： 18
run：second： 6
main： 19
run：first： 7
run：second： 8
run：first： 9
run：second： 10
run：first： 11
run：second： 12
run：first： 13
run：second： 14
run：first： 15
run：second： 16
run：first： 17
run：second： 18
run：first： 19
```

运行结果中两个子线程的 i 变量是连续的，也就是采用实现`java.lang.Runnable`接口方式创建的多个线程可以共享线程类的实例变量。这是因为在这种方式下，程序所创建的Runnable对象只是线程的target，而多个线程可以共享同一个target。

### Callable接口

通过实现`java.util.concurrent.Callable<V>`接口创建并启动多线程的步骤如下：
* 定义`java.util.concurrent.Callable<V>`接口的实现类，并重写该接口的`call()`方法，该`call()`方法将作为`线程执行体`，且该`call()`方法有返回值，再创建Callable实现类的实例。从java 8开始，可以直接使用`Lambda`表达式创建Callable对象
* 使用`java.util.concurrent.FutureTask<V>`类来包装Callable对象，该FutureTask对象封装了`Callable`对象中`call()`方法的返回值
* 使用FutureTask对象作为`java.lang.Thread`类的`target`来创建Thread对象
* 调用线程对象的`start()`方法来启动该线程，可通过调用FutureTask对象的`get()`方法来获得子线程执行结束后的返回值

```java
public class ThirdThread implements Callable<Integer> {

	private int i;

	// 重写call()方法，call()方法的方法体就是线程执行体
	@Override
	public Integer call() throws Exception {
		for (; i < 20; i++) {
			// Thread对象的getName()方法返回当前线程的名称
			System.out.println("run：" + Thread.currentThread().getName() + " " + i);
		}
		return i;
	}

	public static void main(String[] args) {

		// 为主线程设置名称，默认名称为main
		Thread.currentThread().setName("main：");

		for (int i = 0; i < 20; i++) {
			// 调用Thread类的currentThread()方法获取当前线程
			System.out.println(Thread.currentThread().getName() + " " + i);

			if (i == 10) {
				Callable<Integer> callable = new ThirdThread();

				// 使用FutureTask类来包装Callable对象，该FutureTask对象封装了Callable对象中call()方法的返回值
				FutureTask<Integer> ft = new FutureTask<Integer>(callable);

				// 创建并启动线程
				Thread thread = new Thread(ft, "thread：");
				thread.start();

				// 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值
				try {
					System.out.println("return：" + ft.get());
				}
				catch (InterruptedException | ExecutionException e) {
					e.printStackTrace();
				}
			}
		}
	}
}
```

* `Integer thread.ThirdThread.call() throws Exception`方法可以有返回值，可以声明抛出异常

Callable接口是java 5新增的接口，而且它不是Runnable接口的子接口，所以Callable对象不能直接作为Thread类的target。而且call()方法还有一个返回值——call()方法并不是直接调用，它是作为线程执行体被调用。
<br>
<br>
java 5提供了`java.util.concurrent.Future<V>`接口来代表`Callable<V>`接口中call()方法的返回值，并为`Future<V>`接口提供了一个`FutureTask<V>`实现类，该实现类实现了`Future<V>`接口，并实现了`Runnable`接口——可以作为Thread类的target。在`Future<V>`接口中定义了如下几个公共方法来控制它关联的Callable任务：
* `boolean java.util.concurrent.FutureTask.cancel(boolean mayInterruptIfRunning)`，试图取消该Future中关联的Callable任务
* `Integer java.util.concurrent.FutureTask.get() throws InterruptedException, ExecutionException`，返回Callable任务中call()方法的返回值，调用该方法将导致程序阻塞，必须等到子线程结束后才会得到返回值
* `Integer java.util.concurrent.FutureTask.get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException`，返回Callable任务中call()方法的返回值，该方法让程序最多阻塞timeout和unit指定的时间，如果经过指定时间后Callable任务依然没有返回值，将会抛出TimeoutException异常
* `boolean java.util.concurrent.FutureTask.isCancelled()`，如果在Callable任务正常完成前被取消，则返回true
* `boolean java.util.concurrent.FutureTask.isDone()`，如果Callable任务已完成，则返回true

Callable接口有泛型限制，Callable接口中的泛型形参类型与call()方法返回值类型相同。而且Callable接口是函数式接口，因此可使用`Lambda`表达式创建Callable对象。
<br>
<br>
上述示例的main()方法中包含了一个循环，当循环变量i == 10时创建并启动一个新线程，运行结果如下所示：

```
main： 0
main： 1
main： 2
main： 3
main： 4
main： 5
main： 6
main： 7
main： 8
main： 9
main： 10
run：thread： 0
run：thread： 1
run：thread： 2
run：thread： 3
run：thread： 4
run：thread： 5
run：thread： 6
run：thread： 7
run：thread： 8
run：thread： 9
run：thread： 10
run：thread： 11
run：thread： 12
run：thread： 13
run：thread： 14
run：thread： 15
run：thread： 16
run：thread： 17
run：thread： 18
run：thread： 19
return：20
main： 11
main： 12
main： 13
main： 14
main： 15
main： 16
main： 17
main： 18
main： 19
```

上述示例中，当i == 10时，程序启动以FutureTask对象为target的线程。程序最后调用FutureTask对象的get()方法来返回call()方法的返回值——该方法将导致主线程被阻塞，直到call()方法结束并返回为止。

### 对比

通过继承Thread类，实现Runnable、Callable接口都可以实现多线程，不过实现Runable、Callable接口的方式基本相同，只是Callable接口中定义的方法有返回值，可以声明抛出异常而已。因此可以将实现Runnable、Callable接口归为一种方式，这种方式与继承Thread类方式间的主要差别如下：
* 实现Runnable、Callable接口方式创建多线程的优缺点：
  1. 线程类只是实现了Runnable或Callable接口，还可以继承其它类
  2. 此种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况
  3. 缺点是编程稍显复杂，如果需要访问当前线程，则必须使用Thread.currentThread()方法
* 继承Thread类方式创建多线程的优缺点：
  1. 缺点，因为线程类已经继承了Thread类，不能再继承其它父类
  2. 优点，编写简单，如果需要访问当前线程，无须使用Thread.currentThread()方法，直接使用this即可获得当前线程

## 生命周期

在线程的生命周期中，它要经过`新建(New)`、`就绪(Runnable)`、`运行(Running)`、`阻塞(Blocked)`和`死亡(Dead)`五种状态。尤其是当线程启动以后，它不可能一直占用着CPU独自运行，CPU需要在多个线程间切换，于是线程状态也会多次在运行、阻塞间切换。

### 新建和就绪

当程序使用new关键字创建了一个线程之后，该线程就处于`新建状态`，此时它和其它的java对象一样，仅仅由java虚拟机为其分配内存，并初始化其成员变量的值。此时的线程对象没有表现出任何线程的动态特征，程序也不会执行线程的线程执行体。
<br>
<br>
当线程对象调用了start()方法之后，该线程就处于`就绪状态`，java虚拟机会为其创建方法调用栈和程序计数器，处于这个状态中的线程并没有开始运行，只是表示该线程可以运行了。至于该线程何时开始运行，取决于JVM里线程调度器的调度。如上述FirstThread示例中，当循环变量i == 10时创建并启动的两个新线程并未立即执行。
<br>
<br>
需要注意的是，启动线程的正确方法是调用Thread对象的start()方法，而不是直接调用run()方法，否则会变成单线程程序。调用了线程的run()方法后，该线程已经不再处于新建状态，不要再次调用线程对象的start()方法，否则将引发异常。

### 运行和阻塞

如果处于就绪状态的线程获得了CPU，开始执行run()方法的线程执行体，则该线程处于`运行状态`，如果计算机只有一个CPU，那么在任何时刻只有一个线程处于运行状态。当然，在一个多处理器的机器上，将会有多个线程`并行`执行；当线程数大于处理器数时，依然会存在多个线程在同一个CPU上轮换的现象。
<br>
<br>
当一个线程开始运行后，它不可能一直处于运行状态，线程在运行过程中需要被中断，目的是使其它线程获得执行的机会，线程调度的细节取决于底层平台所采用的策略。

线程由`运行状态`进入`阻塞状态`的几种情况：
* 线程调用`sleep(long millis)`方法主动放弃所占用的处理器资源
* 线程调用了一个阻塞式I/O方法，在该方法返回之前，该线程被阻塞
* 线程试图获得一个同步监视器，但该同步监视器正被其它线程所持有
* 线程在等待某个`通知(notify)`

当前正在执行的线程被阻塞之后，其它线程就可以获得执行的机会。被阻塞的线程会在合适的时候重新进入`就绪状态`，即被阻塞线程的阻塞解除后，必须重新等待线程调度器再次调度它。

线程由`阻塞状态`重新进入`就绪状态`的几种情况：
* 调用`sleep(long millis)`方法的线程过了指定时间
* 线程调用的阻塞式I/O方法已经返回
* 线程成功的获得了试图取得的同步监视器
* 线程正在等待某个`通知(notify)`时，其它线程发出了一个`通知(notify)`

![](/image/2017/2017-06-19-java-multithreading-1.png)

线程的状态转换如上图所示，线程从阻塞状态只能进入就绪状态，无法直接进入运行状态。而就绪和运行状态之间的转换通常不受程序控制，是由系统线程调度所决定，当处于就绪状态的线程获得处理器资源时，该线程进入运行状态；当处于运行状态的线程失去处理器资源时，该线程进入就绪状态。但有一个方法例外，调用`yield()`方法可以让运行状态的线程转入就绪状态。

### 死亡

线程会以如下三种方式结束，结束后就处于`死亡状态`：
* run()或call()方法执行完成，线程正常结束
* 线程抛出一个未捕获的异常或错误

当主线程结束时，其它线程不受任何影响，并不会随之结束。一旦子线程启动起来后，它就拥有和主线程相同的地位，不会受主线程的影响。
<br>
<br>
测试线程是否已经死亡时，可以调用线程对象的`boolean java.lang.Thread.isAlive()`方法，当线程处于`就绪`、`运行`、`阻塞`三种状态时，该方法将返回true；当线程处于`新建`、`死亡`两种状态时，该方法将返回false。
<br>
<br>
不要对处于死亡状态的线程调用start()方法，程序只能对新建状态的线程调用start()方法，对新建状态的线程两次调用start()方法也是错误的，这都会引发`java.lang.IllegalThreadStateException`异常，示例如下：

```java
public class StartDead extends Thread {

	private int i;

	@Override
	public void run() {
		for (; i < 20; i++) {
			// 当线程类继承Thread类时，直接使用this即可获取当前线程
			// Thread对象的getName()方法返回当前线程的名称
			System.out.println("run：" + getName() + " " + i);
		}
	}

	public static void main(String[] args) {
		// 为主线程设置名称，默认名称为main
		Thread.currentThread().setName("main：");

		for (int i = 0; i < 20; i++) {
			// 调用Thread类的currentThread()方法获取当前线程
			System.out.println(Thread.currentThread().getName() + " " + i);

			if (i == 10) {
				// 创建并启动第一个线程
				StartDead sd = new StartDead();
				sd.setName("first：");
				// 判断启动后线程的isAlive()值
				System.out.println(sd.isAlive());

				sd.start();

				// 判断启动后线程的isAlive()值
				System.out.println(sd.isAlive());
				// 再次调用start()，java.lang.IllegalThreadStateException异常
				sd.start();
			}
		}
	}

}
```

```
main： 0
main： 1
main： 2
main： 3
main： 4
main： 5
main： 6
main： 7
main： 8
main： 9
main： 10
false
true
run：first： 0
run：first： 1
Exception in thread "main：" run：first： 2
run：first： 3
run：first： 4
run：first： 5
run：first： 6
run：first： 7
run：first： 8
run：first： 9
run：first： 10
run：first： 11
run：first： 12
java.lang.IllegalThreadStateExceptionrun：first： 13

run：first： 14
run：first： 15
	at java.lang.Thread.start(Thread.java:705)run：first： 16

run：first： 17
	at thread.StartDead.main(StartDead.java:36)
run：first： 18
run：first： 19
```

## 控制线程

### join()

`java.lang.Thread`类提供了让一个线程等待另一个线程完成的方法——`void java.lang.Thread.join() throws InterruptedException`
方法，当在某个程序执行流中调用其他线程的join()方法时，调用线程将被阻塞，直到被join()方法加入的join线程执行完为止。
<br>
<br>
join()方法通常由使用线程的程序调用，以将大问题划分为许多小问题，每个小问题分配一个线程。当所有的小问题都得到处理后，再调用主线程来进一步操作，示例如下：

```java
public class JoinThread extends Thread {

	// 提供一个有参数的构造器，用于设置该线程的名字
	public JoinThread(String name) {
		super(name);
	}

	// 重写run()方法，定义线程执行体
	@Override
	public void run() {
		for (int i = 0; i < 20; i++) {
			System.out.println("run：" + getName() + " " + i);
		}
	}

	public static void main(String[] args) throws InterruptedException {
		// 创建并启动第一个子线程
		JoinThread firstThread = new JoinThread("first：");
		firstThread.start();

		for (int i = 0; i < 20; i++) {
			if (i == 10) {
				// 创建并启动第二个子线程
				JoinThread jt = new JoinThread("join：");
				jt.start();

				// main线程调用了jt线程的join()方法，main线程必须等jt执行结束才会向下执行
				jt.join();
			}
			System.out.println(Thread.currentThread().getName() + " " + i);
		}
	}

}
```

```
main 0
run：first： 0
main 1
run：first： 1
run：first： 2
main 2
run：first： 3
run：first： 4
main 3
run：first： 5
main 4
run：first： 6
main 5
run：first： 7
main 6
run：first： 8
main 7
run：first： 9
main 8
run：first： 10
main 9
run：first： 11
run：first： 12
run：first： 13
run：first： 14
run：first： 15
run：first： 16
run：first： 17
run：join： 0
run：first： 18
run：join： 1
run：first： 19
run：join： 2
run：join： 3
run：join： 4
run：join： 5
run：join： 6
run：join： 7
run：join： 8
run：join： 9
run：join： 10
run：join： 11
run：join： 12
run：join： 13
run：join： 14
run：join： 15
run：join： 16
run：join： 17
run：join： 18
run：join： 19
main 10
main 11
main 12
main 13
main 14
main 15
main 16
main 17
main 18
main 19
```

上述示例中共有三个线程，主方法开始时就启动了`first：`子线程，该子线程将会和main线程并发执行。当主线程的循环变量i == 10时启动`join：`线程，该线程不会和main线程并发执行，
main线程必须等该线程执行结束后才可以向下执行。在`join：`线程执行时，实际上只有两个子线程并发执行，而主线程处于阻塞状态，直到`join：`线程执行完成。

### 后台线程

`后台线程/守护线程/精灵线程 (Daemon Thread)`，在后台运行，它的任务是为其它线程提供服务，例如JVM的垃圾回收线程。通过调用Thread对象的`void java.lang.Thread.setDaemon(boolean on)`方法可以将指定线程设置为后台线程，如果所有前台线程死亡，后台线程会自动死亡。

```java
public class DaemonThread extends Thread {

	// 定义后台线程的线程执行体与普通线程没有任何区别
	@Override
	public void run() {
		for (int i = 0; i < 50; i++) {
			System.out.println("run：" + getName() + " " + i);
		}
	}

	public static void main(String[] args) {
		DaemonThread dt = new DaemonThread();
		dt.setName("first：");
		// 将此线程设置成后台线程
		dt.setDaemon(true);
		// 启动后台线程
		dt.start();

		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread().getName() + " " + i);
		}
		// 程序执行到此结束，前台线程（main）结束，后台线程也应该随之结束
	}

}
```

```
run：first： 0
main 0
run：first： 1
main 1
run：first： 2
main 2
run：first： 3
main 3
main 4
run：first： 4
main 5
run：first： 5
main 6
run：first： 6
main 7
run：first： 7
main 8
run：first： 8
main 9
run：first： 9
run：first： 10
run：first： 11
run：first： 12
run：first： 13
run：first： 14
run：first： 15
run：first： 16
run：first： 17
run：first： 18
run：first： 19
run：first： 20
```

上述示例中，本来`first：`线程会执行到i = 49才会结束，但当主线程结束后，JVM会主动退出，因而后台线程也被结束。要将某个线程设置为后台线程，必须在该线程启动之前设置，即setDaemon()必须在start()方法之前调用，否则引发异常。

### sleep()

如果需要让当前正在执行的线程暂停一段时间，并进入阻塞状态，可以通过调用Thread类的静态`sleep(long millis)`方法实现。
<br>
<br>
当前线程调用`sleep(long millis)`方法进入`阻塞状态`后，在其睡眠时间段内，该线程不会获得执行的机会，即使系统中没有其它可执行的线程，处于sleep()中的线程也不会获得执行，因此sleep()方法常用来暂停程序的执行。

```java
public class SleepTest {

	public static void main(String[] args) throws InterruptedException {
		for (int i = 0; i < 5; i++) {
			System.out.println("当前时间:" + new Date());

			// 调用sleep()方法让当前线程(主线程)暂停2s
			Thread.sleep(2000);
		}
	}

}
```

```
当前时间:Sun Jun 25 10:43:59 CST 2017
当前时间:Sun Jun 25 10:44:01 CST 2017
当前时间:Sun Jun 25 10:44:03 CST 2017
当前时间:Sun Jun 25 10:44:05 CST 2017
当前时间:Sun Jun 25 10:44:07 CST 2017
```

## 线程同步

使用两个线程模拟两个人使用同一个账户并发取钱的问题，忽略检查账户和密码的操作，示例如下：

```java
public class Account {

	private String accountNo;// 账户编号

	private double balance;// 账户余额

	public Account() {
	}

	public Account(String accountNo, double balance) {
		this.accountNo = accountNo;
		this.balance = balance;
	}

	public String getAccountNo() {
		return accountNo;
	}

	public void setAccountNo(String accountNo) {
		this.accountNo = accountNo;
	}

	public double getBalance() {
		return balance;
	}

	public void setBalance(double balance) {
		this.balance = balance;
	}

}
```

```java
public class DrawThread extends Thread {

	private Account account;// 用户账户

	private double drawAmount;// 取钱金额

	public DrawThread(String name, Account account, double drawAmount) {
		super(name);// 线程名称
		this.account = account;
		this.drawAmount = drawAmount;
	}

	// 当多个线程修改同一个共享数据时，将涉及数据安全问题
	@Override
	public void run() {
		// 账户余额大于取钱数目时
		if (account.getBalance() >= drawAmount) {
			System.out.println(getName() + "取钱成功，吐出钞票：" + drawAmount);

			// 修改余额
			account.setBalance(account.getBalance() - drawAmount);
			System.out.println("余额为：" + account.getBalance());
		}
		else {
			System.out.println(getName() + "取钱失败，余额不足.");
		}
	}

}
```

```java
public class DrawTest {

	public static void main(String[] args) {
		// 创建一个账户
		Account account = new Account("2017", 1000);

		// 模拟两个线程对同一个账户取钱
		new DrawThread("张晓明", account, 800).start();
		new DrawThread("张大明", account, 800).start();
	}
}
```

```
张晓明取钱成功，吐出钞票：800.0
张大明取钱成功，吐出钞票：800.0
余额为：200.0
余额为：-600.0
```

账户余额只有1000时取出了1600，而且账户余额出现了负值，这是多线程编程容易出现的错误——因为线程调度的不确定性。

### 同步代码块

为解决上述示例中的错误，java的多线程编程引入了`同步监视器`，使用同步监视器的通用方法就是`同步代码块`，`同步代码块`的语法格式如下所示：

```java
synchronized (obj) {
  ...// 此处的代码就是同步代码块
}
```

上面语法格式中的obj就是同步监视器，`线程开始执行同步代码块之前，必须先获得对同步监视器的锁定`。
<br>
<br>
`任何时刻只能有一个线程可以获得对同步监视器的锁定，当同步代码块执行完成后，该线程会释放对该同步监视器的锁定`，通常推荐使用可能被并发访问的共享资源充当同步监视器，使用同步代码块的DrawThread类如下所示：

```java
public class DrawThread extends Thread {

	private Account account;// 用户账户

	private double drawAmount;// 取钱金额

	public DrawThread(String name, Account account, double drawAmount) {
		super(name);// 线程名称
		this.account = account;
		this.drawAmount = drawAmount;
	}

	// 当多个线程修改同一个共享数据时，将涉及数据安全问题
	@Override
	public void run() {
		// 加锁->修改->释放锁
		synchronized (account) {
			// 账户余额大于取钱数目时
			if (account.getBalance() >= drawAmount) {
				System.out.println(getName() + "取钱成功，吐出钞票：" + drawAmount);

				// 修改余额
				account.setBalance(account.getBalance() - drawAmount);
				System.out.println("余额为：" + account.getBalance());
			}
			else {
				System.out.println(getName() + "取钱失败，余额不足.");
			}
		}// 同步代码块结束，该线程释放同步锁
	}

}
```

```
张晓明取钱成功，吐出钞票：800.0
余额为：200.0
张大明取钱失败，余额不足.
```

### 同步方法

与同步代码块对应，java的多线程安全支持还提供了同步方法，同步方法就是使用`synchronized`关键字来修饰某个方法，则该方法成为`同步方法`。通过使用同步方法可以非常方便的实现线程安全的类，线程安全的类具有如下特征：
* 该类的对象可以被多个线程安全的访问
* 每个线程调用该对象的任意方法之后都将得到正确结果
* 每个线程调用该对象的任意方法之后，该对象状态依然保持合理状态

使用同步方法将上述示例改写，如下所示：

```java
public class Account {

	private String accountNo;// 账户编号

	private double balance;// 账户余额

	public Account() {
	}

	public Account(String accountNo, double balance) {
		this.accountNo = accountNo;
		this.balance = balance;
	}

	public String getAccountNo() {
		return accountNo;
	}

	public void setAccountNo(String accountNo) {
		this.accountNo = accountNo;
	}

	public double getBalance() {
		return balance;
	}

	public void setBalance(double balance) {
		this.balance = balance;
	}

	public synchronized void draw(double drawAmount) {
		// 账户余额大于取钱数目时
		if (balance >= drawAmount) {
			System.out.println(Thread.currentThread().getName() + "取钱成功，吐出钞票：" + drawAmount);

			try {
				Thread.sleep(1);
			}
			catch (InterruptedException e) {
				e.printStackTrace();
			}

			// 修改余额
			balance -= drawAmount;
			System.out.println("余额为：" + balance);
		}
		else {
			System.out.println(Thread.currentThread().getName() + "取钱失败，余额不足.");
		}
	}

}
```

```java
public class SyndrawThread extends Thread {

	private Account account;// 用户账户

	private double drawAmount;// 取钱金额

	public SyndrawThread(String name, Account account, double drawAmount) {
		super(name);// 线程名称
		this.account = account;
		this.drawAmount = drawAmount;
	}

	@Override
	public void run() {
		account.draw(drawAmount);
	}

}
```

```java
public class SyndrawTest {

	public static void main(String[] args) {
		// 创建一个账户
		Account account = new Account("2017", 1000);

		// 模拟两个线程对同一个账户取钱
		new SyndrawThread("张晓明", account, 800).start();
		new SyndrawThread("张大明", account, 800).start();
	}
}
```

```
张晓明取钱成功，吐出钞票：800.0
余额为：200.0
张大明取钱失败，余额不足.
```

可变类的线程安全是以降低程序的运行效率作为代价，为了减少线程安全所带来的负面影响，程序可以采用如下策略：
* 不要对线程安全类的所有方法都进行同步，只对那些会改变共享资源的方法进行同步
* 如果可变类有两种运行环境：单线程环境、多线程环境，则应该为该可变类提供两种版本，即线程不安全版本和线程安全版本。在单线程环境中使用线程不安全版本以保证性能，在多线程环境中使用线程安全版本

### 释放同步监视器的锁定

程序会在如下几种情况下释放对同步监视器的锁定：
* 当前线程的同步方法、同步代码块执行结束，当前线程即释放同步监视器
* 当前线程在同步代码块、同步方法中遇到break、return终止了该代码块、该方法的继续执行，当前线程将会释放同步监视器
* 当前线程在同步代码块、同步方法中出现了未处理的错误或异常，导致了该代码块、该方法异常结束时，当前线程将会释放同步监视器
* 当前线程执行同步代码块或同步方法时，程序执行了同步监视器对象的`wait()`方法，则当前线程暂停，并释放同步监视器

在如下所示的情况下，线程不会释放同步监视器：
* 线程执行同步代码块或同步方法时，程序调用`sleep()`、`yield()`方法来暂停当前线程的执行，当前线程不会释放同步监视器

### 同步锁

从java 5开始，java提供了一种功能更强大的线程同步机制——通过显式定义`同步锁`对象来实现同步，在这种机制下，同步锁由Lock对象充当。
<br>
<br>
Lock是控制多个线程对共享资源进行访问的工具。通常，锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前应先获得Lock对象。
<br>
<br>
某些锁可能允许对共享资源的并发访问，如ReadWriteLock(读写锁)，Lock、ReadWriteLock是java 5提供的两个根接口，并为Lock提供了ReentrantLock(可重入锁)实现类，为ReadWriteLock提供了ReentrantReadWriteLock实现类。
<br>
<br>
在实现线程安全的控制中，比较常用的是ReentrantLock，使用该Lock对象可以显式的加锁、释放锁，通常使用ReentrantLock的方式如下：

```java
public class AccountLock {

	// 定义锁对象
	private final ReentrantLock lock = new ReentrantLock();

	private String accountNo;// 账户编号

	private double balance;// 余额

	public AccountLock(String accountNo, double balance) {
		this.accountNo = accountNo;
		this.balance = balance;
	}

	public String getAccountNo() {
		return accountNo;
	}

	public void setAccountNo(String accountNo) {
		this.accountNo = accountNo;
	}

	public double getBalance() {
		return balance;
	}

	// 提供一个线程安全的draw()方法来完成取钱操作
	public void draw(double drawAmount) {
		// 加锁
		lock.lock();

		try {
			// 账户余额大于取钱数目
			if (balance >= drawAmount) {
				System.out.println(Thread.currentThread().getName() + "取钱成功，吐出钞票：" + drawAmount);

				try {
					Thread.sleep(1);
				}
				catch (InterruptedException e) {
					e.printStackTrace();
				}

				balance -= drawAmount;
				System.out.println("余额为：" + balance);
			}
			else {
				System.out.println(Thread.currentThread().getName() + "取钱失败，余额不足.");
			}
		}
		finally {
			// 修改完成，释放锁
			lock.unlock();
		}
	}

}
```

```java
public class AccountLockThread extends Thread {

	private AccountLock accountLock;

	private double drawAmount;

	public AccountLockThread(String name, AccountLock accountLock, double drawAmount) {
		super(name);
		this.accountLock = accountLock;
		this.drawAmount = drawAmount;
	}

	public double getDrawAmount() {
		return drawAmount;
	}

	public void setDrawAmount(double drawAmount) {
		this.drawAmount = drawAmount;
	}

	@Override
	public void run() {
		accountLock.draw(drawAmount);
	}

}
```

```java
public class AccountLockTest {

	public static void main(String[] args) {
		AccountLock al = new AccountLock("2016", 1000);
		new AccountLockThread("刘忠", al, 600).start();
		new AccountLockThread("张忠", al, 600).start();
	}

}
```

```
刘忠取钱成功，吐出钞票：600.0
余额为：400.0
张忠取钱失败，余额不足.
```

使用Lock与使用同步方法类似，只是使用Lock时显式使用Lock对象作为同步锁，而使用同步方法时系统隐式使用当前对象作为同步监视器，都符合`加锁->修改->释放锁`的操作模式。

### 死锁

当两个线程相互等待对方释放同步监视器时就会发生死锁，java虚拟机没有监测，也没有采取措施来处理死锁情况，所以多线程编程时应该采取措施避免死锁出现。`一旦出现死锁，整个程序既不会发生任何异常，也不会给出任何提示，只是所有线程处于阻塞状态，无法继续`，示例如下：

```java
//当两个线程相互等待对方释放同步监视器时就会发生死锁
public class DeadLock implements Runnable {

	A a = new A();

	B b = new B();

	public void init() {
		Thread.currentThread().setName("主线程");

		// 调用A对象的foo方法
		a.foo(b);
		System.out.println("进入了主线程之后");
	}

	@Override
	public void run() {
		Thread.currentThread().setName("副线程");

		// 调用b对象的bar方法
		b.bar(a);
		System.out.println("进入副线程之后");
	}

	public static void main(String[] args) {
		DeadLock d1 = new DeadLock();

		// 以d1为target启动新线程
		new Thread(d1).start();

		// 调用init方法
		d1.init();
	}

}

class A {

	public synchronized void foo(B b) {
		System.out.println("当前线程名:" + Thread.currentThread().getName() + " 进入A实例的foo()方法");
		try {
			Thread.sleep(200);
		}
		catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("当前线程名:" + Thread.currentThread().getName() + " 企图调用B实例的last()方法");
		b.last();
	}

	public synchronized void last() {
		System.out.println("进入A类的last()方法内部");
	}
}

class B {
	public synchronized void bar(A a) {
		System.out.println("当前线程名:" + Thread.currentThread().getName() + " 进入B实例的bar()方法");
		try {
			Thread.sleep(200);
		}
		catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("当前线程名:" + Thread.currentThread().getName() + " 企图调用A实例的last()方法");
		a.last();
	}

	public synchronized void last() {
		System.out.println("进入B类的last()方法内部");
	}
}
```

```
当前线程名:副线程 进入B实例的bar()方法
当前线程名:主线程 进入A实例的foo()方法
当前线程名:副线程 企图调用A实例的last()方法
当前线程名:主线程 企图调用B实例的last()方法
```

## 线程池

系统启动一个新线程的成本是比较高的，因为它涉及与操作系统交互。在这种情况下，使用线程池可以很好的提高性能。
<br>
<br>
从java 5开始，java内建支持线程池，java 5新增了一个`java.util.concurrent.Executors`工厂类来产生线程池，该工厂类包含如下几个静态工厂方法来创建线程池：
* `ExecutorService newCachedThreadPool()`，创建一个具有缓存功能的线程池，系统根据需要创建线程，这些线程将会被缓存在线程池中
* `ExecutorService newFixedThreadPool(int nThreads)`，创建一个可重用的、具有固定线程数的线程池
* `ExecutorService newSingleThreadExecutor()`，创建一个只有单线程的线程池，它相当于调用newFixedThreadPool(int nThreads)方法时传入参数为1 
* `ScheduledExecutorService newScheduledThreadPool(int corePoolSize)`，创建具有指定线程数的线程池，它可以在指定延迟后执行线程任务
* `ExecutorService newSingleThreadExecutor()`，创建只有一个线程的线程池，它可以在指定延迟后执行线程任务

返回的`ExecutorService`对象代表一个线程池，它可以执行Runnable对象或Callable对象所代表的线程；返回的`ScheduledExecutorService`对象也代表一个线程池，它是`ExecutorService`的子类，它可以在指定延迟后执行线程任务。
<br>
<br>
`ExecutorService`代表尽快执行线程的线程池（只要池中有空闲连接就立即执行线程任务），程序只要将一个Runnable对象或Callable对象（代表线程任务）提交给该线程池，该线程池就会尽快执行该任务。ExecutorService中提供了如下几个方法：
* Future<?> submit(Runnable task)：将一个Runnable对象提交给指定的线程池，线程池将在有空闲线程时执行Runnable对象代表的任务
* Future<T> submit(Callable<T> task)：将一个Callable对象提交给指定的线程池，线程池将在有空闲线程时执行Callable对象代表的任务

用完一个线程池后，应该调用该线程池的`shutdown()`方法，该方法将启动线程池的关闭序列，调用`shutdown()`方法后的线程池不再接收新任务，但会将以前所有已提交任务执行完成。当线程池中的所有任务都执行完成后，池中的所有线程都会死亡。
<br>
<br>
使用线程池来执行线程任务的步骤如下：
1. 调用Executors类的静态工厂方法创建一个ExecutorService对象，该对象代表一个线程池
2. 创建Runnable或Callable接口实现类的实例，作为线程执行任务
3. 调用ExecutorService对象的submit()方法来提交Runnable实例或Callable实例
4. 当不想提交任何任务时，调用ExecutorService对象的shutdown()方法来关闭线程池

```java
public class ThreadPoolTest implements Runnable {

	@Override
	public void run() {
		// 局部变量，多个线程不共享；共享实例变量
		for (int i = 0; i < 20; i++) {
			System.out.println("run：" + Thread.currentThread().getName() + " " + i);
		}
	}

	public static void main(String[] args) {
		// 创建一个具有固定线程数（6）的线程池
		ExecutorService pool = Executors.newFixedThreadPool(6);

		ThreadPoolTest tpt = new ThreadPoolTest();
		// 向线程池中提交三个线程
		pool.submit(tpt);
		pool.submit(tpt);
		pool.submit(tpt);

		// 关闭线程池
		pool.shutdown();
	}

}
```

```
run：pool-1-thread-3 0
run：pool-1-thread-3 1
run：pool-1-thread-3 2
run：pool-1-thread-3 3
run：pool-1-thread-3 4
run：pool-1-thread-1 0
run：pool-1-thread-2 0
run：pool-1-thread-1 1
run：pool-1-thread-1 2
run：pool-1-thread-3 5
run：pool-1-thread-1 3
run：pool-1-thread-2 1
run：pool-1-thread-1 4
run：pool-1-thread-3 6
run：pool-1-thread-1 5
run：pool-1-thread-2 2
run：pool-1-thread-1 6
run：pool-1-thread-3 7
run：pool-1-thread-1 7
run：pool-1-thread-2 3
run：pool-1-thread-1 8
run：pool-1-thread-3 8
run：pool-1-thread-1 9
run：pool-1-thread-2 4
run：pool-1-thread-1 10
run：pool-1-thread-3 9
run：pool-1-thread-1 11
run：pool-1-thread-2 5
run：pool-1-thread-1 12
run：pool-1-thread-3 10
run：pool-1-thread-1 13
run：pool-1-thread-2 6
run：pool-1-thread-1 14
run：pool-1-thread-3 11
run：pool-1-thread-2 7
run：pool-1-thread-3 12
run：pool-1-thread-3 13
run：pool-1-thread-2 8
run：pool-1-thread-3 14
run：pool-1-thread-2 9
run：pool-1-thread-2 10
run：pool-1-thread-2 11
run：pool-1-thread-2 12
run：pool-1-thread-2 13
run：pool-1-thread-2 14
```

## 线程相关类

### ThreadLocal

`java.lang.ThreadLocal<T> (Thread Local Variable，线程局部变量)`类，为每一个使用该变量的线程都提供一个变量值的副本，使每一个线程都可以独立的改变自己的副本，而不会和其他线程的副本冲突。从线程的角度看，就好像每一个线程都完全拥有该变量一样。
<br>
<br>
通过使用ThreadLocal类可以简化多线程编程时的并发访问，可以很简捷的隔离多线程程序的竞争资源，它提供如下几个方法：
* T get()：返回此线程局部变量中当前线程副本的值
* void remove()：删除此线程局部变量中当前线程的值
* void set(T value)：设置此线程局部变量中当前线程副本的值

```java
package threadlocal;

public class ThreadLocalTest {

	public static void main(String[] args) {
		// 为主线程设置名称，默认名称为main
		Thread.currentThread().setName("main：");

		Account at = new Account("王初始");

		for (int i = 0; i < 20; i++) {
			if (i == 6) {
				at.setName(Thread.currentThread().getName());
			}
			System.out.println(at.getName() + " 账户的i值：" + i);
		}

		// 启动两个子线程，共享同一个Account
		new MyTest(at, "first：").start();
		new MyTest(at, "second：").start();

	}

}

class Account {

	// 定义一个ThreadLocal类型的变量，该变量将是一个线程局部变量，每个线程都会保留该变量的一个副本
	private ThreadLocal<String> name = new ThreadLocal<String>();

	// 定义一个初始化的name成员变量的构造器
	public Account(String str) {
		this.name.set(str);
		System.out.println("---" + this.name.get());
	}

	public String getName() {
		return name.get();
	}

	public void setName(String str) {
		this.name.set(str);
	}

}

class MyTest extends Thread {

	// 定义一个Account类型的成员变量
	private Account account;

	public MyTest(Account account, String name) {
		super(name);
		this.account = account;
	}

	@Override
	public void run() {
		for (int i = 0; i < 20; i++) {
			if (i == 6) {
				account.setName(Thread.currentThread().getName());
			}
			System.out.println(account.getName() + " 账户的i值：" + i);
		}
	}
}
```

```
---王初始               //主线程访问帐户名有值
王初始 账户的i值：0
王初始 账户的i值：1
王初始 账户的i值：2
王初始 账户的i值：3
王初始 账户的i值：4
王初始 账户的i值：5
main： 账户的i值：6
main： 账户的i值：7
main： 账户的i值：8
main： 账户的i值：9
main： 账户的i值：10
main： 账户的i值：11
main： 账户的i值：12
main： 账户的i值：13
main： 账户的i值：14
main： 账户的i值：15
main： 账户的i值：16
main： 账户的i值：17
main： 账户的i值：18
main： 账户的i值：19
null 账户的i值：0       //第一个子线程启动后帐户名为null，该帐户名是一个副本
null 账户的i值：1
null 账户的i值：2
null 账户的i值：3
null 账户的i值：0       //第二个子线程启动后帐户名为null，该帐户名是一个副本
null 账户的i值：1
null 账户的i值：2
null 账户的i值：3
null 账户的i值：4
null 账户的i值：5
null 账户的i值：4
null 账户的i值：5
second： 账户的i值：6    //第二个子线程在i == 6后帐户名有值
first： 账户的i值：6     //第一个子线程在i == 6后帐户名有值
first： 账户的i值：7
first： 账户的i值：8
second： 账户的i值：7
second： 账户的i值：8
second： 账户的i值：9
second： 账户的i值：10
second： 账户的i值：11
second： 账户的i值：12
second： 账户的i值：13
second： 账户的i值：14
second： 账户的i值：15
second： 账户的i值：16
second： 账户的i值：17
second： 账户的i值：18
second： 账户的i值：19
first： 账户的i值：9
first： 账户的i值：10
first： 账户的i值：11
first： 账户的i值：12
first： 账户的i值：13
first： 账户的i值：14
first： 账户的i值：15
first： 账户的i值：16
first： 账户的i值：17
first： 账户的i值：18
first： 账户的i值：19
```

上述示例中，帐户名有三个副本，主线程一个，另外启动的两个子线程各一个，它们的值互不干扰，每个线程完全拥有自己的ThreadLocal变量，这就是ThreadLocal的用途。
<br>
<br>
ThreadLocal和其它所有的同步机制一样，都是为了解决多线程中对同一变量的访问冲突。在普通的同步机制中，是通过对象加锁来实现多个线程对同一资源的安全访问，该资源是多个线程共享的，系统并没有将这份资源复制多份，只是采用了安全机制来控制对这份资源的访问而已。ThreadLocal从另一个角度来解决多线程的并发访问，它将需要并发访问的资源复制多份，每个线程拥有一份资源，每个线程拥有自己的资源副本，也就没有必要对该变量进行同步了。
<br>
<br>
ThreadLocal并不能替代同步机制，两者面向的问题领域不同。同步机制是为了同步多个线程对相同资源的并发访问，是多个线程之间进行通信的有效方式；而ThreadLocal是为了隔离多个线程的数据共享，从根本上避免多个线程之间对共享资源的竞争，也就不需要对多个线程进行同步了。
<br>
<br>
通常建议：`如果多个线程间需要共享资源，以达到线程间的通信功能，就使用同步机制；如果仅仅需要隔离多个线程间的共享冲突，则可以使用ThreadLocal`。

### 线程安全的集合类

ArrayList、LinkedList、HashSet、TreeSet、HashMap、TreeMap等都是线程不安全的，当多个并发线程向这些集合中存、取数据时，可能会破坏这些集合的数据完整性。从java 5开始，在`java.util.concurrent`包下提供了大量支持高效并发访问的集合接口和实现类，如下图所示：

![](/image/2017/2017-06-19-java-multithreading-2.png)

上图中，这些线程安全的集合类可分为如下两类：
* 以Concurrent开头的集合类，如ConcurrentHashMap
* 以CopyOnWrite开头的集合类，如CopyOnWriteArrayList

其中以Concurrent开头的集合类代表了支持并发访问的集合，它们可以支持多个线程并发写入访问，这些写入线程的所有操作都是线程安全的，但读取操作不必锁定。

* [本文所用的完整项目代码托管在github](https://github.com/soyuone/thread.git)
* [参考：《疯狂java讲义》]()