JDK9之后，HotSpot所有功能的日志都收归到了“-Xlog”参数上

```java
-Xlog[:[selector][:[output][:[decorators][:output-options]]]]

selector:选择器，由标签Tag和日志级别Level共同组成（标签可以理解为迅即某个功能模块的名字）
    日志级别从低到高，Trace，Debug，Info，Warning，Error，Off六种级别
    decorator修饰器可以要求每行日志输出都附加上额外的内容，支持附加在日志上的信息包括：
        ·time：当前日期和时间。 
        ·uptime：虚拟机启动到现在经过的时间，以秒为单位。 
        ·timemillis：当前时间的毫秒数，相当于System.currentTimeMillis()的输出。 
        ·uptimemillis：虚拟机启动到现在经过的毫秒数。
        ·timenanos：当前时间的纳秒数，相当于System.nanoTime()的输出。
        ·uptimenanos：虚拟机启动到现在经过的纳秒数。
        ·pid：进程ID。
        ·tid：线程ID。
        ·level：日志级别。
        ·tags：日志输出的标签集。
   不指定的话，默认值是uptime、level、tags这三个
12345678910111213141516
```

1、查看GC日志基本信息：jdk9之前用-XX:+PrintGC，jdk9之后用-Xlog:gc
2、查看GC详细信息：jdk9之前用-XX:+PrintGCDetails，jdk9之后用-Xlog:gc*
3、查看GC前后的堆、方法区可用容量变化：jdk9之前用-XX:+PrintHeapAtGC，jdk9之后用-Xlog:gc+heap=debug
4、查看GC过程中用户线程并发时间以及停顿的时间:在jdk9之前用-XX:+Print- GCApplicationConcurrentTime以及-XX:+PrintGCApplicationStoppedTime，jdk9之后用-Xlog:safepoint
5、查看收集器Ergonomics机制（自动设置堆空间各分代区域大小、收集目标等内容，从Parallel收集器开始支持）自动调节的相关信息。在JDK 9之前用-XX：+PrintAdaptiveSizePolicy，JDK 9之后用-Xlog：gc+ergo*=trace
6、查看熬过收集后剩余对象的年龄分布信息，在JDK 9前使用-XX：+PrintTenuring-Distribution， JDK 9之后使用-Xlog：gc+age=trace



## 参考资料

OpenJDK 11 JVM 日志相关参数解析与使用：https://gitchat.blog.csdn.net/article/details/104764605

