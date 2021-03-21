# Java12新特性

美国当地时间2019年3月19日，北京时间3月20日，Java 12发布！

Java 12总共有8个新的JEP（JDK Enhancement Proposals）

http://openjdk.java.net/projects/jdk/12/

**Feature**

- 189 : [Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)](http://openjdk.java.net/jeps/189)
- 230 : [Microbenchmark Suite](http://openjdk.java.net/jeps/230)
- 325 : [Switch Expressions (Preview)](http://openjdk.java.net/jeps/325)
- 334 : [JVM Constants API](http://openjdk.java.net/jeps/334)
- 340 : [One AArch64 Port, Not Two](http://openjdk.java.net/jeps/340)
- 341 : [Default CDS Archives](http://openjdk.java.net/jeps/341)
- 344 : [Abortable Mixed Collections for G1](http://openjdk.java.net/jeps/344)
- 346 : [Promptly Return Unused Committed Memory from G1](http://openjdk.java.net/jeps/346)



### Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)

Shenandoah垃圾回收器是Red Hat在2014年宣布进行的一项垃圾收集器研究项目Pauseless GC的实现，旨在针对JVM上的内存回收实现低停顿的需求。该设计将与应用程序线程并发，通过交换CPU并发周期和空间以改善停顿时间，使得垃圾回收器执行线程能够在Java线程运行时进行堆压缩，并且标记和整理能够同时进行，因此避免了在大多数JVM垃圾收集器中所遇到的问题。

据Red Hat研发Shenandoah团队对外宣传，Shenandoah垃圾回收器的暂停时间与堆大小无关，这意味着无论将对设计为200MB还是200GB，都将拥有一致的系统暂停时间，不过实际使用性能将取决于实际工作堆的大小和工作负载。

与其他Pauseless GC类似，Shenandoah GC主要目标是99,9%的暂停小于10ms，暂停与堆大小无关。

Shenandoah是第一款不是由Oracle（包括以前的Sun）公司的虚拟机团队所领导开发的HotSpot垃圾收集器。因此受到了来自“官方”的排挤，直至OracleJDK 15，Oracle仍然明确拒绝支持Shenandoah收集器，并执意在打包OracleJDK时通过条件编译完全排除掉了Shenandoah的代码。换句话说，Shenandoah是一款只有OpenJDK才会包含的垃圾收集器。

虽然Shenandoah只在OpenJDK中才有，但是在[OpenJDK官网](http://openjdk.java.net/)下载的JDK中实际上是不包含Shenandoah的。JDK有不同的发型版本，其中AdoptOpenJDK包含有Shenandoah。如果要使用Shenandoah垃圾收集器就需要使用AdoptOpenJDK。

> AdoptOpenJDK是OpendJDK的社区维护版，主要维护8,11两个LTS版本以及最新版本。
>
> Official：https://adoptopenjdk.net/





### Microbenchmark Suite





### Switch Expressions (Preview)











1、switch表达式

2、Shenandoah GC：低停顿时间的GC

3、JVM常量API

4、微基准测试套件

5、只保留一个AArch64实现

6、默认生成类数据共享（CDS）归档文件

7、可中断的G1 Mixed GC

8、增强G1，自动返回未使用的堆内存给操作系统







如何评估一款GC的性能

- 吞吐量：程序的运行时间（程序的运行时间+内存回收的时间）
- 垃圾收集开销：吞吐量的补数，垃圾收集器所占时间与总时间的比例。
- 暂停时间：执行垃圾收集时，程序的工作线程被暂停的时间。
- 收集频率：相对于应用程序的执行，收集操作发生的频率。
- 堆空间：Java堆区所占的内存大小。
- 快速：一个对象从诞生到被回收所经历的时间。

Node：垃圾收集器中吞吐量和低延迟这两个目标是相互矛盾的，因为如果选择以吞吐量优先，那么必然需要降低内存回收的执行频率，但是这样会导致GC需要更长的暂停时间来执行内存回收。相反的。如果选择以低延迟优先为原则，那么为了降低每次执行内存回收时的暂停时间，也只能频繁地执行内存回收，但这又引起了年轻代内存的缩减和导致程序吞吐量的下降。









学习资料：[bilibili - Java12＆13新特性](https://www.bilibili.com/video/BV1sJ411M7JA?p=8)



