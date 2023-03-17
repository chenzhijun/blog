---
title: 查漏补缺 Java 系列 - JVM 基础概念
copyright: true
date: 2021-04-04 15:15:49
tags: Java
categories: Java
---

# 查漏补缺 Java 系列 - JVM

你知道 JVM 吗?作为Java开发人员，我想你肯定听过吧?那么你了解为什么需要 JVM 吗?它的作用是什么?给我带来了什么好处，又给我们带来了什么缺点了?接下来我们一起看看吧。

## JVM 的基础概念

什么是 JVM ? 

JVM(Java Virtual Machine) 中文名称为 Java 虚拟机。它的作用是为 Java 提供跨平台运行的基础，也就是那句"一次编译，到处运行"的底层保证。 

我想你肯定在 windows，Linux，MacOS 上都有运行过 Java 程序，那么你想一想，在一台裸机上面(指新买的机器装上一个纯的干净的系统 OS)，你直接编写一个 Main。java 能运行起来么?运行一个 Main。java，我想你肯定经过这几个步骤:安装 jdk(jre) -> 编写 Main。java -> `javac Main。java`生成`main。claas`文件 -> `java Main`;那你想过吗?为什么你要安装一个 jkd(jre)，为什么你要先用`javac`命令，生成一个`claas`文件，然后才能执行了?

你肯定有疑问。是的。我想跟你说的是其实 jvm 根本不知道你写的 java 文件里面的内容，一点都不知道。但是它认识你按照它的规范生成的 class 文件。然后它再将 class 文件翻译成机器识别的机器码。这样才能真正的运行你的程序了。也许这就是为什么很多人会说它是解释器吧。
<!--more-->
## JVM 都含有什么?

我们知道 java 代码要在 jvm 里面运行的话，上面的概念解释中也说了，它需要由 java 编译器生成 class 字节码文件，然后再由 jvm 将字节码 class 文件翻译成底层不同的平台所能识别的机器码。那么在读取到解释成为字节码的过程中，它肯定需要进行一些操作，它进行操作肯定也就避免不了要使用一些工具（或者我们称之为资源）。它要用到的资源就是内存。就像我们住在自己的房子里面一样，我们肯定会要定下来，哪里是我们的卧室，哪里是我们的厨房，哪里是我们的客厅，哪个地方是书房等等。JVM 也是一样。JVM 将它所管理的内存分为了几个运行时数据区域。那么 JVM 有哪几个区域呢？看 [Run-Time Data Areas](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-2.html#jvms-2.5) 总共是 5 个运行时数据区：程序计数器（PC register），Java 虚拟机栈（Java Virtual Machine Stacks），堆（Heap），方法区（Method Area），运行时常量池（Run-Time Constant Pool），本地方法栈（Native Method Stacks）。

1. 程序计数器（PC register）：线程私有；jvm 中每一个线程都会有的一块区域，如果是非本地方法（not native thread）的线程，会保存一个当前指令的执行地址；如果是本地方法的线程（native thread）,这个位置就为空（undefined）；这块的空间足够大，大到足够支持所有的指令地址和本地方法指针，所以这块不会出现 OOM 的时候。（原话：The Java Virtual Machine's pc register is wide enough to hold a returnAddress or a native pointer on the specific platform.）

2. Java 虚拟机栈（Java Virtual Machine Stacks）：线程私有；每个线程创建的时候同步创建这块区域与线程的生命周期相同；它的主要主要作用是用来存储[栈帧](https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.6)（Frame）；它存储局部变量，操作数栈，动态链接，方法调用，方法返回值等信息；Java 虚拟机栈的空间分配可以是不连续的一段内存空间；jvm 规范规定这块的空间可以是固定大小，也可以是动态的大小，由具体的虚拟机实现。在 Java 虚拟机栈中，如果一个运行中的线程请求比 jvm 允许的栈大小更大的空间，jvm 就会直接抛出`StackOverflowError`；如果是动态扩容机制，当本地内存不足的时候，会抛出`OutOfMemoryError`。

3. 堆（Heap）：是一块线程共享的区域，在 java 虚拟机启动的时候就会被创建，主要作用是用来储存对象实例和数组。这块区域可以被垃圾回收器回收，如果需要的资源比垃圾回收期回收后释放出来的资源还要大，那它就会抛出`OutOfMemoryError`。实际上，这块区域可以设置为动态扩缩容，应该提供设置最大堆内存和最小堆内存的方法。这块区域的空间申请可以是不连续的空间。

4. 方法区（Method Area）：方法区是一个线程共享区域，在 java 虚拟机启动的时候会被创建，主要用来存储每个类的结构，比如说类信息，常量，静态变量，方法和构造方法等。逻辑上它是堆（Heap）的一个区域，但是简单实现它可以不需要使用垃圾回收期回收它。如果需要动态调节它的大小，应该提供最大空间和最小空间的设定方法。如果方法区的内存不够分配了，应该要抛出`OutOfMemoryError`异常。

5. 运行时常量池（Run-Time Constant Pool）：属于方法区的一部分，在类或者接口创建的时候生成，主要用来存储编译期间的各种字面量和符号引用。运行时常量池对于 Class 文件有动态性，不要求常量一定只有编译期才能产生，也可以在运行期加入新的常量，比如使用 String.intern（）方法。如果该区的空间不够，那么就会跟方法区一样抛出`OutOfMemoryError`异常。

6. 本地方法栈（Native Method Stacks）：这个有点类似于 Java 虚拟机栈，虚拟机栈是为执行 Java 方法服务，本地方法栈则为调用 native 方法服务，也会抛出`StackOverflowError`,`OutOfMemoryError`异常。

7. 直接内存/堆外内存：我在官网的介绍中其实并没有看到这个，但是我看书（《深入理解 Java 虚拟机上》）里面介绍，就一起放到这里了。这块的空间其实就是系统内存中除了分配给 jvm 那些数据区域的，剩下的内存。这块的空间受限于机器本身的内存总大小。这块的作用其实就类似于 NIO 中的 channel 直接操作 Native 函数库直接分配堆外内存。

> ps 方法区很多人也叫永久代（permanent Generation），本质上两者并不等价，这只是 hotspot 虚拟机的一种具体实现而已，用永久代来实现方法区。书是基于 jdk1.7 的，1.8 中 PermGen 已经被去掉了[whats-new-on-jdk-8](https://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)。

现在你知道了上面定义的 6 个内存 jvm 运行区域，但是在各家的 jvm 虚拟机实现中有点不同。比如 HotSpot，其实我们可以看到只要有上面的规定的 6 个运行时区就行了，毕竟这是规范，具体实现肯定不同：

![2020-04-04-23-12-46](/images/qiniu/2020-04-04-23-12-46.png)

了解这么多的内存区域，那么它是怎么识别一个对象是否应该被回收？怎么回收空间的了？用到了什么算法？
明天继续！

参考资料：

[jvm](https://blog.csdn.net/zhenghongcs/article/details/104628800)

[Java Language and Virtual Machine Specifications](https://docs.oracle.com/javase/specs/index.html)

[gc 调优-官网文档](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/index.html)