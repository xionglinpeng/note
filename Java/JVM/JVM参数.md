- `-Xms`：
- `-Xmx`：
- `-Xmn`：
- `-XX:Survivor-Retio`：
- `-verbose:gc`：
- 
- `-XX:+UseSerialGC`：启动serial收集器
- `-XX:+UseParNewGC`：启用ParNew收集器
- `-XX:+UseParallelGC`：启用Parallel收集器
- `-XX:+UseParallelOldGC`：启用Parallel Old收集器
- `-XX:+UseConcMarkSweepGC`：启用CMS垃圾收集器
- `-XX:+UseG1GC`：启用G1垃圾收集器
- `-XX:+UseZGC`：启用ZGC垃圾收集器
- `-XX:+UseShenandoahGC`：启用shenanndoah垃圾收集器
- 
- `-XX:PretenureSizeThreshold`：指定大于该设置值得对象直接在老年代分配。此参数只对`Serial`和`ParNew`两款新生代收集器有效。
- `-XX:MaxTenuringThreshold`：
- `-XX:HandlePromotionFailure`：设置值是否允许担保失败；如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升老年代对象的平均大小，如果大于，将尝试进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者`-XX:HandlePromotionFailure`设置不允许冒险，那这时就改为进行一次Full GC。

**类加载**

- `-XX:-UseSplitVerifier`：关闭类加载-验证-方法体校验阶段`StackMapTable`优化。
- `-XX:+FailOverToOldVerifier`：要求在类型校验失败的时候退回到旧的类型推导方式进行校验（JD7+（主版本号>50）此参数已无效）。
- `-Xverify:none`：关闭类加载阶段Class二进制数据校验。

**日志**

- `-Xlog:gc` and `-XX:+PrintGCDetails`：发生垃圾收集行为时打印内存回收日志，并在进程退出的时候输出当前的内存个区域分配情况。

- `-Xlog:gc*`：输出GC详细日志
- `-Xloggc:[/path/to/gc.log]`：将GC日志输出到`/path/to/gc.log`文件中`-Xlog:gc`

`-XX:GCLogFileSize`设置合适的GC日志文件大小，使用`-XX:NumberOfGCLogFiles`设置要保留的GC日志文件个数，使用`-Xloggc:/path/to/gc.log`设置GC日志文件的位置



- `-XX:+DisableExplicitGC`：禁用`Runtime.getRuntime().gc()`和`System.gc()`；



**对象**

- `-XX:-UseCompressedOops`：启用/禁用对象头的类型指针压缩。