# Garbage-First(G1) Garbage Collector

## Introduction to Garbage-First (G1) Garbage Collector

G1垃圾收集器主要是为那些拥有**大内存**的**多核处理器**而设计的。它在以很高的概率满足垃圾收集的停顿时间要求，同时还可以达到很高的吞吐量，同时几乎不需要做什么配置。G1的目标是**为应用提供停顿时间和吞吐量的最佳平衡**，它的主要特性包含：

- 堆内存达到数十个G甚至更大，超过50%的堆内存都是存活的对象。

- 对象分配和晋升的速度随时间变化非常大。

- 堆中存在大量的内存碎片。

- 可预测的停顿时间的目标在几百毫秒以内，不会存在长时间的停顿。

在JDK11中，G1已经取代了CMS。至JDK9开始，G1就是默认的垃圾收集器。

## Enabling G1

因为G1是默认的收集器，因此一般不需要做任何额外的操作就可以开启。你可以使用`-XX:+UseG1GC`来明确开启。

## Basic Concepts

G1是一款分代、增量、并行、主要是并发、stop-the-world、标记整理的垃圾收集器，在每一个STW停顿的时间，它都会监控停顿时间的目标。跟其他的收集器类似，G1把对分成逻辑上的yong区和old区。内存回收主要集中在yong区，在这个区域的内存回收也是非常高效的，偶尔也会发生在old区。

为了提高吞吐量，有些操作总是STW，还有一些操作在应用停止的情况下会花费更多的时间，比如一些对整个堆的操作，像全局的标记就是并行和并发来执行的，在空间回收的时候，为了让STW时间更短，G1是增量的分步和并行来回收的。G1是通过记录上一次应用的行为和GC的停顿信息来实现可预计的停顿时间的，可以利用这些信息来计算在停顿时间之内要做的工作的多少。比如：G1会首先回收那些可以高效回收的内存区域（也就是大部分都被填满垃圾的区域，这也是为什么叫G1的原因）。

G1主要是通过evacuation（疏散）来回收空间：将选定区域中存活的对象复制到新的区域，并在此过程中对其进行压缩，evacuation（疏散）完成后，先前存活对象所在区域将清空并重新用于应用程序分配。

G1不是实时收集器，长时间来看，有较高的概率可以满足设定的停顿时间的目标，但这并不是绝对的。

### Heap Layout

G1将堆划分为多个大小相等的堆区域（region），每个region的连续范围如下图所示。Region是内存分配和内存回收的基本单位。在任何时刻，这些region都可能是空的（浅灰色），或分配了特定的世代，yong或者old。当对内存进行分配时，内存管理器将释放可用的region。内存管理器为空闲的region分配对应的世代，然后将它们返回给应用程序进行内存分配。

![](https://docs.oracle.com/en/java/javase/15/gctuning/img/jsgct_dt_004_grbg_frst_hp.png)

年轻代包含eden区（红色）和survivor区（红色+'S'）。这些区域与其他收集器中相应的连续空间所不同的是，在G1中，这些区域以**非连续**的方式存在堆内存中。Old区（浅蓝色）构成了老年代。老年代区域可能是大（humongous ）对象区（浅蓝色+‘H’），横跨多个region。

应用程序总是优先将对象分配给年轻代，即eden区。但humongous对象除外，它将直接分配到老年代。

### Garbage Collection Cycle

宏观上看，G1的垃圾回收在两个阶段中来回交替执行。young-only阶段会逐步把old区填满存回对象，space-reclamation阶段除了会回收young区的内存以外，还会增量回收old区的内存。然后就会重新开启young-only阶段。

*有关此循环的概述：*

![Description of Figure 7-2 follows](https://docs.oracle.com/en/java/javase/15/gctuning/img/jsgct_dt_001_grbgcltncyl.png)

222

1. Young-only阶段：这个阶段从young区的一些普通的yong GC开始，这些GC会把yong区的对象晋升到old区。当old区的占有率达到一个阈值时，就开始young-only阶段到space-reclamation阶段的转换，这个时候，就开始Concurrent Start的yong GC，而不是普通的yong GC。
   - Concurrent Start : 这种类型的GC除了做普通的yong GC之外，还会开始标记（marking）过程。并发标记会找出当前old区中所有存活的对象以备随后的space-reclamation阶段来使用。当并发标记还没有结束的过程中，还是可能会发生普通的yong GC。标记经过两个特殊的STW停顿之后才会结束：重新标记（remark）和清理（cleanup）。
   - Remark: 这个停顿中会结束掉mark阶段，做**全局的引用处理**和**类卸载**，**回收全空的region**和**清理掉内部的数据结构**。在重新标记（remark）和清理（cleanup）之前，G1会并发的在选中的old区的region中计算随后可以回收的空间，这个计算会在cleanup的时候结束。
   - Cleanup: 这个停顿中会决定是否要开始space-reclamation阶段。如果要开始space-reclamation，young-only阶段就结束了，紧接着就开始一个Mixed Young GC。
2. Space-reclamation阶段：这个阶段由多个Mixed GC组成，不光是回收young区的region，同时也会回收old区的region。当G1发现，无法回收更多old区的region时，space-reclamation阶段就结束了。

一个回收周期完成以后，会从另一个young-only阶段开始一个新的回收周期。在垃圾回收时，如果内存不足，G1也会像其他收集器一样STW，做整个堆的压缩（也就是Full GC）。

> old区的占有率达到一个阈值时，这个阈值是多少？
>
> 那些情况会导致内存不足？

### Garbage Collection Pauses and Collection Set





## Garbage-First Internals

本节描述了一些Garbage-First (G1) 垃圾收集器的重要细节。

### Java Heap Sizing

G1在调整Java堆大小时遵循标准规则，使用`-XX:InitialHeapSize`设置Java堆最小大小，`-XX:MaxHeapSize`设置Java堆最大大小，`-XX:MinHeapFreeRatio`表示可用内存最小百分比，`-XX:MaxHeapFreeRatio`表示可用内存最大百分比。G1收集器考虑在`Remark`期间调整Java堆的大小，`Full GC`仅暂停。此过程可能会向操作系统释放内存或从操作系统中分配内存。

#### Young-Only Phase Generation Sizing

G1总是在正常yong集合结束是为下一个mutator阶段调整年轻代的大小。这样，G1可以根据对实际暂停时间的长期观察满足`-XX:MaxGCPauseTimeMillis`和`-XX:PauseTimeIntervalMillis`设定的暂停时间目标。它考虑了同样大小的年轻代需要多长时间才能evacuate。这包括在收集期间需要复制多少对象，以及这些对象之间的互连程度等信息。

如果没有其他的约束，那么G1自适应地在`-XX:G1NewSizePercent`和`-XX:G1MaxNewSizePercent`确定的值之前调整年轻代的大小以满足暂停时间。

或者，`-XX:NewSize`和`-XX:MaxNewSize`可以分别用来设置最小和最大的年轻代大小。

> Note：仅仅指定`-XX:NewSize`和`-XX:MaxNewSize`可以精确地将年轻代的大小固定为其设定的值。这将禁用暂停时间的控制。

#### Space-Reclamation Phase Generation Sizing

在space-reclamation阶段，G1会尽量在一个GC停顿期间回收尽可能多的老年代的内存。年轻代的大小将被设置为允许的最小值，这个值由`-XX:G1NewSizePercent`选项确定。

在此阶段的每个混合（Mixed）收集开始时，G1都会从候选回收集中选择一组region，以添加到回收集中（CSet）。

- 老年代区域最小集合，以确保疏散进度。这个老年代区域的集合是由候选回收集中region的数量除以Space-Reclamation阶段的长度（由`-XX:G1MixedGCCountTarget`确定）确定。
- 如果G1预测回收最小集合之后还有剩余时间，则从候选回收集中添加额外的老年代region。添加老年代region，直到预计使用剩余时间的80%为止。
- 一组可选的回收集region集合，G1在其他两部分被疏散，并且在当前停顿中还有剩余时间之后逐步疏散。



空间的剩余数量，在其候选回收集中可回收的region小于`-XX:G1HeapWastePercent`设定的百分比时，Space-Reclamation阶段结束。

只要是存在可回收内存的old区的region都会被添加到回收集合中，一直到再增加就会超过停顿时间的目标为止。在特定的某个GC停顿之内，G1会按照这些region回收的效率来添加要回收的region，效率高的排在前面，得到最终要回收的region集合。

每一个GC停顿要回收的old区的region数量受限于候选region集合数量除以`-XX:G1MixedGCCountTarget`这个选项设置的长度，候选region集合中old区的所有占用率低于``的那些region。

当候选region集合中可回收的空间低于`-XX:G1HeapWastePercent`的时候，这一阶段就结束了。

### Periodic Garbage Collections





### Determining Initiating Heap Occupancy

*Initiating Heap Occupancy Percent (IHOP)*是触发开始初始标记的阈值，它的值是old区大小的一个百分比。通过观察标记过程使用的时间和标记过程中old区分配的内存大小，G1默认会自动的调整一个最优的IHOP，这个特性叫做自适应的IHOP。如果激活了这个特性，`-XX:InitiatingHeapOccupancyPercent`选项决定了IHOP的初始值，因此初始的时候还没有足够的数据来对这个值做预测。`-XX:-G1UseAdaptiveIHOP`选项可以关闭自适应，此时，由`-XX:InitiatingHeapOccupancyPercent`设置的阈值就始终不会改变。

自适应IHOP会这样来设置这个阈值：当开始space-reclamation阶段的第一个Mixed GC时，old区的占有率=old区的最大值减去`-XX:G1HeapReservePercent`。

### Marking

G1使用SATB算法来做标记。在初始标记开始的时候，G1会保存堆的一份虚拟镜像，初始标记开始时的存活对象在后续的标记过程中仍然被认为是存活的。这意味着就算是标记过程中这部分对象死亡了，对于space-reclamation阶段来说它们仍然是存活的（有少部分例外）。跟其他的收集器相比，这会导致一些额外的内存被错误的占用。但是，SATB给Remark提供了更低的延迟。那些在mark中死亡的对象在下一次mark中会被回收掉。

### Behavior in Very Tight Heap Situations

当应用的存活对象占用了大量的内存。无法容纳回收剩余的对象的时候，就会发生evacuation失败。evacuation失败发生的时候，G1为了完成当前的GC，它会保持已经位于新的位置上的存活对象，仅仅是调整对象之间的引用，而不会复制或者移动这些对象，evacuation失败会知道一些额外的开销，但是一般会跟别的young GC一样快。evacuation失败完成以后，G1会跟往常一样继续恢复应用的执行。G1会假设evacuation失败时发生在GC的后期，也就是说，大部分对象已经被移动过了，已经有足够的剩余内存来继续执行应用程序一直到mark结束，space-reclamation开始。

如果这个假设不成立，G1最终会发起Full GC。这种类型的GC会对整个堆做压缩，可能会非常的慢！

### Humongous Objects

大对象是指那些对象大小超过region一半大小的对象。如果没有设置`-XX:G1HeapRegionSize`选择，那么当前region的大小是自适应计算的。

G1对这些大对象会做特殊处理：

- 大对象是分配在old区的一系列连续的region中，对象的开始总是位于这一系列region的第一个region的开始，在整个对象被回收掉之前，最后一个region中剩余的空间都不会被分配出去。
- 一般来说，只有在cleanup停顿阶段mark结束以后或者FullGC的时候，死亡的大对象才会被回收掉。但是大对象是基本类型的数组是例外，比如bool数组、所有的整型数组、浮点型数组等。如果它们不在有引用，G1会在任何GC停顿的适当时候回收这些大对象。这个默认是开启的，可以使用选项`-XX:G1EagerReclaimHumongousObjects`关闭。
- 分配大对象会导致过早的发生GC停顿。G1在分配每一个大对象的时候都会去检查IHOP，如果当前的堆占用率超过了IHOP阈值，就会立即强制发起一个初始标记的yong GC。
- 即使在Full GC的时候，这些大对象也是永远不会被移动的。这会导致过早的发生Full GC或者意外的OOM，尽管此时还有大量空闲的内存，但是这些内存都是region中的内存碎片。

## Ergonomic Defaults for G1 GC

本主题概述了特定于G1的最重要的默认值及其默认值。它们给出了使用G1的预期行为和资源使用情况的粗略概述，没有任何附加选项。

| 选项和默认值                                                 | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-XX:MaxGCPauseMillis=200`                                   | 最大停顿时间目标，默认200毫秒                                |
| `-XX:GCPauseTimeInterval`= *<ergo>*                          | 最大停顿时间间隔目标，默认情况下，没有目标，G1允许在极端情况下连续执行垃圾收集。 |
| `-XX:ParallelGCThreads`= *<ergo>*                            |                                                              |
| `-XX:ConcGCThreads`= *<ergo>*                                | 用于并发工作的最大线程数。默认情况下，                       |
| `-XX:+G1UseAdaptiveIHOP`，`-XX:InitiatingHeapOccupancyPercent=45` |                                                              |
| `-XX:G1HeapRegionSize`=*<ergo>*                              | 堆region大小。默认值基于最大堆大小，并计算为，大小必须为2的幂，有效值为1~32。 |
| `-XX:G1NewSizePercent=5`，`-XX:G1MaxNewSizePercent=60`       |                                                              |
| `-XX:G1HeapWastePercent=5`                                   |                                                              |
| `-XX:G1MixedGCCountTarget=8`                                 |                                                              |
| `-XX:G1MixedGCLiveThresholdPercent=85`                       |                                                              |

## Comparison to Other Collectors

G1与其他收集器的主要区别：

- 并行的GC只能从整体上压缩和回收老年代空间，G1将这项工作逐步分配到较短的暂停中，这大大减少了停顿时间，但是潜在地增加了吞吐量。
- G1可以在执行新生代回收的同时执行部分老年代空间的回收。
- 由于并发的原因，G1可能会表现出比其他收集器更高的性能开销，这将影响吞吐量。
- ZGC的目标是非常大的堆内存空间，旨在以更大的吞吐量成本提供更小的暂停时间。

由于其工作方式的不同，G1具有一些独特的特性来提高垃圾收集的效率：

- G1可以在任何垃圾回收期间回收一些完全是空的，较大的region。这样可以避免很多其他不必要的垃圾收集，不需要付出过多代价即可回收大量的空间。
- G1可以尝试同时对Java堆上的重复字符串进行重复数据删除。

默认情况下，G1启用了从老年代回收完全是空的，较大的region。可以使用选项`-XX:-G1EagerReclaimHumongousObjects`禁用此功能。

默认情况下，G1禁用了字符串重复数据删除。可以使用选项`-XX:+G1EnableStringDeduplication`启用此功能。





G1其本质上也是基于分代概念的一款收集器，只是其每种分代的内存空间连续而已。为此，G1的垃圾收集有四个阶段，分别是：

- 新生代GC
- 并发标记周期
- 混合GC
- Full GC

新生代GC：对于新生代GC，只要Eden区空间被填满时，就会启动。但这并不是绝对的，因为在后续的每次并发标记周期和混合GC的同时都会进行新生代GC，而此时，Eden区可能并没有被填满。每一次的新生代GC，都会将Eden空间清空，并将一部分对象复制到Survivor区域以及一部分对象晋升到老年代区域。

并发标记周期：对于并发标记周期，其主要的目的就是并发的去扫描老年代中的垃圾。在这个阶段并不仅仅是扫描，还会清理老年代中的垃圾，不过需要注意的是，此阶段的清理仅仅是清理region全空，以及大对象的区域，简而言之，就是这个region中全是垃圾。至于其他region，则需要留到混合GC阶段区回收。在官方文档中，描述了两种情况会触发并发标记周期，第一种情况是old区占用整堆的45%（此比例可调整）；第二种情况是每次分配大对象。实际上还有一种情况没有说明，即元空间不足扩容也会触发并发标记周期。

混合GC：顾名思义，就是新生代的垃圾回收和老年代的垃圾回收会同时进行，新生代的垃圾回收跟其他阶段的新生代GC没有区别，只是在选择回收集时候会额外的添加老年代的region到回收集中，而其他阶段不会。在混合GC阶段，回收老年代的时候仅仅只需要添加老年代的region到回收集中即可，因为老年代中存在哪些垃圾，已经在并发标记周期完成了。混合GC会连续多次进行，因为考虑到停顿限制的问题，G1并不会在混合GC阶段一次对所有老年代的垃圾进行全量回收，而是在考虑到不超过限制停顿时间的前提下，进行增量地依次回收部分老年代region。





1. 验证Mixed回收
2. 新生代三种concurrent start模式：区域触发，45%触发，源空间触发





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