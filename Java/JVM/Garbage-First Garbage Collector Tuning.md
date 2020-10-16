# Garbage-First Garbage Collector Tuning

## G1通用推荐设置

G1一般推荐使用它的默认设置，然后设置一个停顿时间和最大堆内存的目标。

G1跟别的收集器不一样，G1默认配置的目标即不是最大化吞吐量也不是最小化停顿时间，而是使用时间相对较短的停顿来达到很高的吞吐量。但是，G1的这种增量回收内存和停顿时间的控制机制不管是对应于线程还是对内存回收的效率都会导致一些额外的开销。

如果你更想要高吞吐量，那就设置相对宽松的停顿时间的目标（`-XX:MaxGCPauseMillis`）或者提供更大的堆内存。如果停顿时间更重要，那就修改停顿时间的目标。要避免使用`-Xmn`,` -XX:NewRatio`这样的参数来限制yong区的大小，因为G1主要是靠自适应调整yong区的大小来满足停顿时间的目标的。如果设置yong区的大小，那就会覆盖掉并且会禁用掉对停顿时间的控制。

## 从其他收集器切换到G1

一般来说，当从其他收集器切换到G1的时候，比如CMS，首先删掉所有影响GC的参数，只设置停顿时间的目标和整个堆的大小（使用`-Xmx`和可选的`-Xms`）。

许多对其他收集器在某些方面有用的参数，对G1要么就没作用，要么甚至会降低吞吐量来满足停顿时间的目标。比如如果设置了yong区的大小会完全阻止G1调整yong区的大小来满足停顿时间的目标。

## 提高G1的性能

G1被设计成不用做额外的设置就可以提供整体上很好的性能。但是，有时候这种默认的行为和配置并不能提供最优的结果。本节就对如何诊断和改善这些情况给出了一些指导意见。这些指导意见可能只是在特定应用的某个纬度上改善GC性能。具体情况要具体分析，应用程序级别的优化可能比虚拟机的优化带来更高的效果，比如说减少长期存活的对象可以避免很多不确定的情况出现。

为了方便诊断，G1提供了大量的日志。一般要从设置`-Xlog:gc*=debug`开始，如果需要的话再细化输出结果。日志提供了详细的GC过程和停顿的信息，包括GC的类型、在停顿的每个阶段所花费的时间等等。

下面的章节描述了一些常见的性能问题。

## Full GC调优

Full GC一般是非常耗时的。old区堆占有率高导致的Full GC在日志中通过*Pause Full (Allocation Failure)*这样的字符就可以发现，Full GC一般发生在带有`to-space exhausted`标记的evacuation failure。

一种Full GC是因为应用分配了太多的对象来不及快速回收导致的，这种情况下并发标记还没结束来不及到space-reclamation阶段。Full GC还有可能是由分配很多大对象引起的。基于这些对象在G1中的分配方式，它们会占用比预期大小大很多的内存。

所以调优目标就是确保并发标记可以按时结束。要么降低old区内存分配速度，要么给并发标记更多的执行时间都可以达到这个目标。

G1提供了好几个参数来更好的处理这种情况：

- 通过`gc+heap=info`这种日志可以发现堆中大对象占用的region数量，日志中这一行`"Humongous regions: X->Y"`中的Y就是大对象占用的region数量。如果相对old区总的region数来说，这个数值相对偏高，那最好就要减少这个数值。可以使用`-XX:G1HeapRegionSize`调大region的大小来减少大对象占用的region数。当前的region的大小在日志开头有输出。
- 增大堆的大小。一般也会增加mark的时间。
- 增加并发标记的线程数，通过`-XX:ConcGCThreads`这个参数。
- 强制G1提早开始mark。G1会根据之前的应用的行为来自动决定IHOP的阈值。如果应用的行为变化了，先前的预测就是错误的。可以有两种方式：通过增加自适应IHOP计算的`-XX:G1ReservePercent`的值来减少堆的目标占有率提前开始space-reclamation阶段，或者通过`-XX:-G1UseAdaptiveIHOP`禁用掉自适应IHOP，然后使用`-XX:InitiatingHeapOccupancyPercent`手动设置IHOP。

其他Full GC一般是应用程序或者是外部的工具导致的。如果是由`System.gc()`导致的，并且还没有办法修改源代码，可以使用`-XX:+ExplicitGCInvokesConcurrent`来减轻Full GC的可能性，或者是使用`-XX:+DisableExplicitGC`然后虚拟机完全忽略掉这句代码。外部的工具也可能会导致Full GC，如果不要就可以删掉它们。

## 大对象碎片调优

在堆内存耗尽之前，为了找到一系列连续的region也会导致Full GC。这种情况可以通过设置`-XX:G1HeapRegionSize`增大region的大小减少大对象的region数量，或者是增加整个堆的大小。极限情况下，虽然看上去还有很多可用内存，但是G1找不到足够的连续的内存来分配对象。如果Full GC也无法回收足够的可用连续内存就会导致虚拟机退出。这种情况下，只能要么较少大对象的数量，要么就增加堆的大小。

## 停顿时间调优

本节讨论如何改善停顿时间太长的问题。

### System Time和Real-Time

在每一个GC停顿的时候，日志中的`gc+cpu=info`会包含一行停顿时间是如何话费的信息，比如：`User=0.19s Sys=0.00s Real=0.01s`。

User time是虚拟机花费的时间，System time是操作系统花费的时间，Real time是停顿花费的绝对时间。如果System time相对较高，很可能是环境的原因。

System time高导致的几个问题：

- 虚拟机从操作系统那里申请和归还内存会导致不必要的延迟。可以使用`-Xms`和` -Xmx`把虚拟机的最大最小内存设置为同一个值来避免这种延迟，还可以使用`-XX:+AlwaysPreTouch`来预分配所有内存，这样把延迟提前到了虚拟机启动阶段。
- 尤其是在Linux中，通过THP技术把小分页合并成一个大分页可能会拖慢任意一个进程，而不仅仅是在停顿的时候。因为虚拟机会分配和占用大量的内存，所以很可能会被操作系统选中而被拖慢很长时间。如果来禁用THP要参考你的操作系统手册。
- 写日志也会拖慢一些时间，因为会有后台的间断的任务占用了写日志的磁盘的IO带宽。可以考虑给你的日志使用单独的磁盘或者其他的存储，比如使用内存文件系统。

另一种需要注意的情况是System time比其他两个的总和还要长，这说明虚拟机没有得到足够的CPU，主机的负载可能太高了。

### 引用对象处理时间太长

在引用处理阶段（Reference Processing）会展示引用对象处理花费的时间信息。在引用处理阶段，G1会根据引用对象的类型来更新应用对象的引用。默认，G1使用下面的启发式算法并行化引用处理的各个子阶段：对每一个引用对象开启一个单独的线程，这个值受限于`-XX:ParallelGCThreads`。可以把`-XX:ReferencesPerThread`设置为0，这样就会使用所有的可用线程，或者使用`-XX:-ParallelRefProcEnabled`完全禁用掉并行处理。

### Young-Only阶段时间太长

Young GC花费的时间一般跟young区的大小成正比例关系，更确切的说跟young区的回收集合中要拷贝的存活对象的数量有关。如果这个时间太长，假如是对象拷贝时间长那就减少`-XX:G1NewSizePercent`。这会减少young区的最小大小，会减小停顿时间。

如果手动设置young区的大小带来别的问题，如果应用的性能尤其是存活对象的数量突然发生了变化，这会导致停顿时间出现毛刺，通过设置`-XX:G1MaxNewSizePercent`减小最大young区的大小可以有效的减轻这个问题。这个参数限制了young区的最大尺寸，因此也限制了停顿时间内需要处理的对象数量。

### Mixed GC时间太长

Mixed GC主要是用来回收old区的内存。Mixed GC的回收集合包含了young区和old区的region，启用日志中的`gc+ergo+cset=trace`，就可以得到young区和old区的回收对停顿时间分别做了多大贡献的信息。分别对照下young区和old区的可预测的停顿时间。

如果是young区时间太长，可以参考*Young-Only阶段时间太长*。否则，就要减少old区的停顿时间，G1提供了下面3个选项：

- 增大`-XX:G1MixedGCCountTarget`把old区的回收分步到更多次的GC当中。
- 设置`-XX:G1MixedGCLiveThresholdPercent`避免把那些回收时间长的region加入到回收集合。很多时候，存活对象占有率高的region会话费更多的时间来回收。
- 可以让Space-Reclamation尽早结束，这样G1就不会收集跟多的占用率高的region。这种情况下可以调大`-XX:G1HeapWastePercent`。

需要注意，后面的2个参数会减少候选集合的数量，这样在一个回收阶段能回收的空间数量也会减少。这意味着G1有可能并不能回收足够的old区空间来做后续的操作，但是随后的Space-Reclamation可能会把这些垃圾回收掉。

### Update RS和Scan RS时间长。

G1为了能回收old区的region，它需要记录跨region的引用关系，也就是从一个region指向另一个region的引用。跨region的引用集合被叫做region的remember set。当移动region的内存的时候，必须要更新RS。对RS的维护几乎是并发的，为了性能的原因，当应用创建了跨region的引用的时候，G1并不会立即就更新RS，更新RS的请求会被推迟并且批量执行。

GC的时候需要一个完整的RS，Update RS阶段就会去处理那些未完成的更新RS的请求，Scan RS阶段就是在RS中查找对象之间的引用，移动region中的对象，更新这些对象的引用到新的位置。这两个阶段可能会花费大量的时间，这取决于应用的不同。

通过设置`-XX:G1HeapRegionSize`来调整region的大小，这会影响跨region的引用数量，同时也会影响RS的大小。处理region的RS可能是GC中花费时间很长的一部分，因此这直接影响了最大停顿时间。含有少量跨region引用的大region在处理RS的时间上会比较少，但是大的region也就意味着region中有更多的存活对象要回收，这会增加其他阶段的时间。

G1并发处理RS更新，Update RS花费的时间大概是最大的停顿时间的`-XX:G1RSetUpdatingPauseTimePercent`，如果减少这个值，G1会并发做的更多的更新RS的工作。

如果伴随着应用分配大对象而造成的假的Update RS时间太长，G1会尝试批处理这些更新。如果创建这样的批处理正好在GC之前，那么GC就需要在Update RS停顿的时间之内处理完所有的这些工作。可以设置`-XX:-ReduceInitialCardMarks`禁用这种行为来避免发生潜在的这种情况。

Scan RS的时间是由G1为了保持RS的数量比较小而做的压缩的数量来决定的。RS在内存中的存储越紧凑，在垃圾回收中要检索出存储的值就会花费越多的时间。当更新RS的时候，根据RS的大小，G1会自动做这样的压缩，这叫做RS coarsening。在最高级别压缩的情况下，检索出实际的数据可能非常的慢。`-XX:G1SummarizeRSetStatsPeriod`这个选项结合日志中的`gc+remset=trace`可以表明是否发生了coarsening。如果发生了coarsening，在*Before GC Summary*这一块*Did <X> coarsenings*这一行中的X会出现一个很高的值，`-XX:G1RSetRegionEntries`这个选项可以极大地减少这些coarsening的发生。在生产环境中要避免使用这个详细的RS日志，因为这些数据会占用大量的时间。

## 吞吐量调优

G1默认在吞吐量和停顿时间保持一个平衡。但是，有的场景是需要高吞吐量的。除了前面章节介绍的如何降低停顿时间，还可以减少停顿的频率，这主要是使用`-XX:MaxGCPauseMillis`来调大停顿时间。分代大小自适应策略会自动的调整young区的大小，这会直接决定停顿的频率。如果达不到预期的效果，尤其是在space-reclamation阶段，可以通过`-XX:G1NewSizePercent`增大最小yong区的内存来强制G1这么做。

代表了young区的最大大小的`-XX:G1MaxNewSizePercent`，有些情况下因为限制了young区的大小因此会限制吞吐量。通过日志中的region summary输出中的`gc+heap=info`可以发现。在这种情况下，Eden区和Survivor区的大小之和接近`-XX:G1MaxNewSizePercent`。这个时候就要考虑调大`-XX:G1MaxNewSizePercent`。

另一个增加吞吐量的办法是减少并发的工作量，比如并发的更新RS经常会需要大量的CPU资源。调大`-XX:G1RSetUpdatingPauseTimePercent`这个值会把并发操作的工作移动到GC停顿中。最坏的情况是并发的更新RS可以被禁用，通过设置这几个参数`-XX:-G1UseAdaptiveConcRefinement` `-XX:G1ConcRefinementGreenZone=2G` `-XX:G1ConcRefinementThreads=0`，这就会禁用这种机制，并且把所有的RS更新操作移动到下一次GC停顿中。

使用`-XX:+UseLargePages`来启用大页内存也可以提高吞吐量。关于如何设置大内存页可以参考你的操作系统的文档。

还可以禁用堆大小来调整最小化这部分工作，可以把`-Xms`和`-Xmx`设置为相同的值，此外，还可以使用`-XX:+AlwaysPreTouch`在虚拟机启动的时候让操作系统使用物理内存来支持虚拟机的虚拟内存（也就是提前把内存分配好）。这些操作都能够让停顿时间跟预期的更一致。

## 堆大小调优

跟其他的收集器类似，G1的目标是让花费在垃圾收集的时间低于`-XX:GCTimeRatio`这个值。可以调整这个值来满足你的需求。

## 默认值调优

本节讲述了关于这个话题的一些默认值和一些命令行参数的额外信息。

| Option and Default Value                                     | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `-XX:+G1UseAdaptiveConcRefinement`,`-XX:G1ConcRefinementGreenZone=`*<ergo>*,`-XX:G1ConcRefinementYellowZone=`*<ergo>*,`-XX:G1ConcRefinementRedZone=`*<ergo>*,`-XX:G1ConcRefinementThreads=`*<ergo>* | 并发RS更新需要这些参数来控制并发线程的工作。这些操作G1都有自适应的默认值，`-XX:G1RSetUpdatingPauseTimePercent`这个值是GC停顿中花费在处理剩余工作的时间，可以按需调整这个值，调整这个值要当心因为可能会引起非常长时间停顿。 |
| `-XX:+ReduceInitialCardMarks`                                | 这个参数把并发RS更新跟新对象分配放到一起执行。               |
| `-XX:+ParallelRefProcEnabled`，`-XX:ReferencesPerThread=1000` | `-XX:ReferencesPerThread`决定了并发度：每N个引用对象都会有一个线程参与引用处理，这个值受限于`-XX:ParallelGCThreads`。如果设置为0，标明最大线程数总是`-XX:ParallelGCThreads`。这个决定了对`java.lang.Ref.*`这些对象实例的处理是否是多个线程并行执行的。 |
| `-XX:G1RSetUpdatingPauseTimePercent=10`                      | 这个值决定了G1花费在更新剩余RS的Update RS阶段上的时间占总的垃圾回收时间的百分比。 |
| `-XX:G1SummarizeRSetStatsPeriod=0`                           | 这个值代表G1产生RS报告的周期，设置为0就禁用。产生RS报告是非常耗时的操作，所以只有必要的时候才可以设置一个相对合理的比较高的值。可以用`gc+remset=trace`来查看。 |
| `-XX:GCTimeRatio=12`                                         | 它是花在垃圾收集的时间和花在应用上的时间的比率的分母。实际花在垃圾收集的时间的计算方式是`1 / (1 + GCTimeRatio)`。默认会有8%的时间花在垃圾收集上。 |
| `-XX:G1PeriodicGCInterval=0`                                 | The interval in ms to check whether G1 should trigger a periodic garbage collection. Set to zero to disable. |
| `-XX:+G1PeriodicGCInvokesConcurrent`                         | If set, periodic garbage collections trigger a concurrent marking or continue the existing collection cycle, otherwise trigger a Full GC. |
| `-XX:G1PeriodicGCSystemLoadThreshold=0.0`                    | Threshold for the current system load as returned by the hosts` getloadavg()` call to determine whether a periodic garbage collection should be triggered. A current system load higher than this value prevents periodic garbage collections. A value of zero indicates that this threshold check is disabled. |

> Note: `<ergo>` means that the actual value is determined ergonomically depending on the environment.

## 参考资料

JDK11-G1收集器调优：https://blog.csdn.net/goldenfish1919/article/details/82924205

