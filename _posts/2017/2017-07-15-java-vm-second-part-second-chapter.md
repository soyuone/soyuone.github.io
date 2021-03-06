---
layout: post
title: "深入理解java虚拟机：java内存区域与内存溢出异常"
categories: java
tags: java 虚拟机
author: Kopite
---

* content
{:toc}


本文系阅读`《深入理解java虚拟机》，周志明著`一书中`第二部分 自动内存管理机制``第二章 java内存区域与内存溢出异常`的笔记。



## 运行时数据区域

java虚拟机在执行java程序的过程中会把它管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，以及创建和销毁的时间，有的区域随着虚拟机进程的启动而存在，有些区域则依赖用户线程的启动和结束而建立和销毁。
<br>
<br>
根据《java虚拟机规范(java SE 7版)》的规定，java虚拟机所管理的内存将会包括以下几个运行时数据区域：

![](\image\2017\2017-07-15-java-vm-second-part-second-chapter-1.png)

* 程序计数器
* java虚拟机栈
* 本地方法栈
* java堆
* 方法区

### 程序计数器

`程序计数器（Program Counter Register）`是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。在虚拟机的概念模型里，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。
<br>
<br>
由于java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，称这类内存区域为`线程私有的内存`。
<br>
<br>
如果线程正在执行的是一个java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Native方法，这个计数器值则为空。
<br>
<br>
`此内存区域是唯一一个在java虚拟机规范中没有规定任何OutOfMemoryError情况的区域`。

### java虚拟机栈

与程序计数器一样，`java虚拟机栈（java Virtual Machine Stacks）`也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是java方法执行的内存模型：每个方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。
<br>
<br>
局部变量表存放了编译器可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，不等同于对象本身，可能是一个指向对象起始地址的引用指针）和returnAddress类型（指向了一条字节码指令的地址）。
<br>
<br>
其中，64位长度的long和double类型数据会占用2个局部变量空间（Slot），其余的数据类型只占用一个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。
<br>
<br>
在java虚拟机规范中，对此区域规定了两种异常状况：
* 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出`StackOverflowError`异常
* 如果虚拟机栈可以动态扩展，一旦扩展时无法申请到足够的内存，就会抛出`OutOfMemoryError`异常

### 本地方法栈

`本地方法栈（Native Method Stack）`与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行java方法（也就是字节码）服务，而本地方法栈则为虚拟机栈使用到的Native方法服务。
<br>
<br>
在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它，甚至有的虚拟机（例如，Sun HotSpot虚拟机）直接就把本地方法栈和虚拟机栈合二为一。
<br>
<br>
与虚拟机栈一样，本地方法栈区域也会抛出`StackOverflowError`和`OutOfMemoryError`异常。

### java堆

对于大多数应用来说，`java堆（java Heap）`是java虚拟机所管理的内存中最大的一块。java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。java虚拟机规范中的描述是：所有的对象实例以及数组都要在堆上分配，但是随着JIT编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化发生，所有的对象都分配在堆上也渐渐变得不是那么绝对了。
<br>
<br>
java堆是垃圾收集器管理的主要区域，因此很多时候也被称做`GC堆(Garbage Collected Heap)`。从内存回收的角度看，由于现在收集器基本都采用分代收集算法，所以java堆中还可以细分为：新生代和老年代，再细致一点的有Edon空间、From Survivor空间，To Survivor空间等。从内存分配的角度来看，线程共享的java堆中可能划分出多个线程私有的分配缓冲区(Thread Local Allocation Buffer)。
<br>
<br>
根据