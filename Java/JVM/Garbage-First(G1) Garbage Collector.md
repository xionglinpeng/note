# Garbage-First(G1) Garbage Collector



## Garbage-First Internals



### Java Heap Sizing

本节描述了一些Garbage-First (G1) 垃圾收集器的重要细节。

#### Young-Only Phase Generation Sizing

G1在调整Java堆大小时遵循标准规则，使用`-XX:InitialHeapSize`设置Java堆最小大小，`-XX:MaxHeapSize`设置Java堆最大大小，`-XX:MinHeapFreeRatio`表示可用内存最小百分比，`-XX:MaxHeapFreeRatio`表示可用内存最大百分比。G1收集器只考虑在`Remark`和`Full GC`暂停期间调整Java堆的大小。这个进程可以向操作系统释放内存，也可以从操作系统分配内存。



### Determining Initiating Heap Occupancy











https://blog.csdn.net/goldenfish1919/article/details/82911948

https://blog.csdn.net/weixin_44988663/article/details/107344471

-Xlog:gc*
-Xlog:gc+heap=debug
-Xmx400M
-Xms400M
-XX:+ExplicitGCInvokesConcurrent
-XX:G1HeapRegionSize=4M
-Xlog:gc:D:\program\Workspace\empty-window\gc.log
-verbose:gc
-XX:+PrintVMOptions
-XX:+PrintCommandLineFlags
-XX:+PrintFlagsFinal
-XX:+UnlockExperimentalVMOptions
-XX:G1NewSizePercent=60
-XX:G1MaxNewSizePercent=60
-XX:-G1UseAdaptiveIHOP



https://www.oschina.net/translate/jdk-9-pitfalls

https://zhuanlan.zhihu.com/p/52841787?from_voters_page=true

https://zhuanlan.zhihu.com/p/54048685