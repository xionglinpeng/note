# JVM





GC Roots:

1. 在虚拟机栈（栈帧中的本地变量表）中引用的对象，比如各个线程被调用的方法堆栈中使用到的参数、局部变量、临时变量等。
2. 在方法区中类静态属性引用的对象，比如Java类的引用类型静态变量。
3. 在方法区中常量引用的对象，比如字符串常量池（String Table）里的引用。
4. 在本地方法栈JNI（即通常所说的Native方法）引用的对象。
5. Java虚拟机内部的引用，如基本数据类型对应的Class对象，一些常驻的异常对象（比如NullPointException、OutOfMemoryError）等，还有系统类加载器。
6. 所有被同步锁（synchronized关键字）持有的对象。
7. 反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等。

## 引用

在JDK1.2版之后，Java对引用的概念进行了扩充，将引用分为强引用（Strongly Reference）、软引用（Soft Reference）、弱引用（Weak Reference）和虚引用（Phantom Reference）4种，这4种引用强度依次组件减弱。

- **强引用**是最传统的“引用”的定义，是指在程序代码之中普遍存在的引用赋值，即类似“Object obj = new Object”这种引用关系。无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象。
- **软引用**是用来描述一些还有用，但非必须的对象。只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。在JDK1.2版之后提供了SoftReference类来实现软引用。
- **弱引用**也是用来描述那些非必须对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生为止。当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在JDK1.2版之后提供了WeakReference类来实现弱引用。
- **虚引用**也称为“幽灵引用”或者“幻影引用”，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象的实例。为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。在JDK1.2版之后提供了PhantomReference类来实现虚引用。

## 生存还是死亡

即使在可达性分析算法中判定为不可达的对象，也不是“非死不可”的，这时候它们暂时还处于“缓刑”阶段，要真正宣告一个对象死亡，至少要经历两次标记过程：

如果对象在进行可达性分析后发现没有与GC Roots相连接的引用链，那么它将会被第一次标记，随后进行一次筛选，筛选的条件是此对象是否有必要执行finalize()方法。假如对象没有覆盖finalize()方法，或者finalize()方法已经被虚拟机调用过，那么吸泥机将这两种情况都视为“没有必要执行”。

如果这个对象被判定为确有必要执行finalize()方法，那么该对象将会被放置一个名为F-Queue的队列之中，并在稍后由一条由虚拟机自动建立的、低调度优先级的Finalizer线程去执行它们的finalize()方法。这里所说的“执行”是指虚拟机会触发这个方法开始运行，但并不承诺一定会等待它运行结束。这样做的原因是，如果某个对象的finalize()方法执行缓慢，或者更极端地发生了死循环，将很可能导致F-Queue队列中的其他对象永久处于等待，甚至导致整个内存回收子系统的崩溃。

finalize()方法是对象逃脱死亡命运的最后一次机会，稍后收集器将对F-Queue中的对象进行第二次小规模的标记，如果对象要在finalize()中成功拯救自己——只要重新与引用链上的任何一个对象建立关联即可。譬如把自己（this关键字）赋值给某个类变量或者对象的成员变量，那在第二次标记时它将被移出“即将回收”的集合；如果对象这时还没有逃脱，那基本上它就真的要被回收了。



## 回收方法区

对方法区的垃圾收集“性价比”比较低：在Java堆中，尤其是新生代中，对常规应用进行一次垃圾收集通常可以回收70%至99%的内存空间，相比之下，方法区回收囿于苛刻的判定条件，其区域垃圾收集的回收成果往往远低于此。

因“性价比”原因，部分垃圾收集器未实现或未能完整实现方法区类型的卸载。

方法区的垃圾收集主要回收两部分内容：

1. 废弃的常量。

   回收废弃的常量与回收Java堆中的对象非常类似。

2. 不在使用的类型。

   判断一个类型是否属于“不再被使用的类”，需要同时满足下面三个条件：

   - 该类的所有实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
- 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如OSGI、JSP的冲加载等，否则通常很难达成。
   - 该类对应的java,lang.Class对象没有任何地方被引用，无法在任何地方通过反射访问改类的方法。

注意：Java虚拟机被允许对满足上述三个条件的无用类进行回收，这里说的仅仅是“被允许”，而并不是和对象一样，没有引用了就必然会被回收。

关于是否要对类型进行回收，HotSport虚拟机提供了`-Xnoclassgc`参数进行控制。

还提供了以下参数可以查看类加载和卸载信息：

```jvm
-verbose:class
-XX:+TraceClass-Loading
-XX:TraceClassUnLoading
```

`-verbose:class`和`-XX:+TraceClass-Loading`可以在Product版的虚拟机中使用。

`-XX:TraceClassUnLoading`参数需要FastDebug版的虚拟机支持。

> 在大量使用反射、动态代理、CGLIB等字节码框架，动态生成JSP以及OSGI这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力。



## 垃圾收集算法



分代收集理论

集中算法思想

发展过程



### 分代收集理论



分代假说：

1. 弱分代假说：绝大多数对象都是朝生夕灭。
2. 强分代假说：熬过越多次垃圾收集过程的对象就越难以消亡。
3. 跨带引用假说：跨带引用相对于同代引用来说仅占极少数。

名词定义：

- 部分收集（Partial GC）:指目标不是完整收集整个Java堆的垃圾收集
  - 新生代收集（Minor GC/Young GC）：指目标只是新生代的垃圾收集。
  - 老年代收集（Major GC/Old GC）：指目标只是老年代的垃圾收集。目前只有CMS收集器会有单独收集老年代的行为。另外请注意“Major GC”这个说法现在有点混淆，在不同资料上常有不同所指，读者需按上下文区分到底指老年代的收集还是整堆收集。
  - 混合收集（Mixed GC）：指目标是收集整个新生代以及部分老年代的垃圾收集。目前只有G1收集器会有这种行为。
- 整堆收集（Full GC）：收集整个Java堆和方法区的垃圾收集。

### 标记-清除算法

算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，也可以反过来，标记存活的对象，统一回收所有未被标记的对象。

缺点：

- 执行效率不稳定，如果Java堆中包含大量对象，而且其中大部分是需要被回收的，这时必须进行大量标记和清除的动作，导致标记和清除两个过程的执行效率都随对象数量的增长而降低。
- 内存空间的碎片化问题，标记、清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致较大对象无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

### 标记-复制算法

将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后在把已使用过的内存空间一次清理掉。

优点：

- 对于多数对象都是可回收的情况，算法需要复制的就是占少数的存活对象，而且每次都是针对整个半区进行内存回收，分配内存时也就不用考虑有空间碎片的复杂情况，只要移动堆顶指针，按顺序分配即可。这样实现简单，运行高效。

缺点：

- 如果内存中多数对象都是存活的，这种算法将会产生大量的内存间复制的开销。
- 可用内存缩小为原来的一半，空间浪费大。

> 现在商用Java虚拟机大多都优先采用了标记-复制算法去回收新生代，IBM公司曾有一项专门研究对新生代“朝生夕灭”的特点做了更量化的诠释——新生代中的对象有98%熬不过第一轮收集，因此并不需要按照1:1的比例来话费新生代的内存空间。

Appel式回收

在1989年，Andrew Appel针对具备“朝生夕灭”特点的对象，提出了一种更优化的半区复制分代策略，现在称为”Appel式回收“。

Appel式回收的具体做法是吧新生代划分为一块较大的Eden空间和两块较小的Survivor空间，每次分配内存只使用Eden和其中一块Survivor空间。发生垃圾收集时，将Eden和Survivor中仍然存活的对象一次性复制到另外一块Survivor空间上，然后直接清理掉Eden和已经用过的那块Survivor空间。当Survivor空间不足以容纳一次Minor GC之后存活的对象时，就需要依赖其他内存区域（实际上大多就是老年代）进行分配担保（Handle Promotion）。

HotSpot虚拟机默认Eden和Survivor的大小比例是8:1，也即每次新生代中可用内存空间为整个新生代容量的90%（Eden的80%加上一个Survivor的10%），只有一个Survivor空间，即10%的新生代是会被“浪费”的。

### 标记-整理算法

算法分为“标记”和“整理”两个阶段：首先标记出所有需要回收的对象，也可以反过来，标记存活的对象（所有未被标记的对象就是需要被回收的对象），然后让所有存活的对象都向内存空间的一端移动，最后直接清理掉边界以外的内存。

标记-清除算法与标记-整理算法的本质差异在于前者是一种非移动式的回收算法，而后者是移动式的。

## HotSpot的算法细节实现



### 根节点枚举



## 经典垃圾收集器

收集器的目标，特性，原理，使用场景



![](https://img2018.cnblogs.com/blog/1217276/201904/1217276-20190419123650113-2038401125.png)

### Serial收集器

**目标：**

Serial收集器是最基础、历史最悠久的收集器，曾经（在JDK1.3.1之前）是HotSpot虚拟机新生代收集器的唯一选择。

**特性：**

- Serial收集器是一个单线程工作的收集器。

- Serial收集器在进行垃圾收集时，必须暂停其他所有工作线程，直到它收集结束。

**原理：**

Serial收集器运行过程：

![](https://img2018.cnblogs.com/blog/1217276/201904/1217276-20190419125749048-1944774651.png)

**优点：**

- Serial收集器相比于其他收集器而言更加简单和高效（与其他收集器的单线程相比）。

- 对于内存资源受限的环境，它是所有收集器里额外内存消耗（Memory Footprint）最小的。

- 对于单核处理器或处理器核心数较少的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集其可以获得最高的单线程收集效率。

**缺点：**

- Serial收集器在进行垃圾收集时，必须暂停其他所有工作线程（Stop The Word）。

**适用场景:**

用户桌面应用场景，部分微服务应用以及客户端模式等应用中，其分配给虚拟机管理的内存一般不会特别大，在这种情况下，Serial收集器是一个很好的选择。

### ParNew收集器

**目标：**

ParNew收集器实质上是Serial收集器的多线程并行版本。用于新生代的多线程收集器。ParNew收集器是JDK7之前的遗留系统中首选的新生代收集器，其中一个与功能、性能无关但是其实很重要的原因是：除了Serial收集器外，目前只有它能与CMS收集器配合工作。

**特性：**

- ParNew收集器是一个多线程工作的收集器。
- ParNew收集器在进行垃圾收集时，必须暂停其他所有工作线程，直到它收集结束。
- ParNew收集器默认开启的收集线程数与处理器核心数量相同。

**原理：**

ParNew收集器运行过程：

![](https://img2018.cnblogs.com/blog/1217276/201904/1217276-20190419132509209-425788073.png)

优点：

- 随着机器的处理器核心数越多，相应的垃圾收集性能以及资源利用率越好。

缺点：

- 多个线程进行垃圾收集，存在线程切换开销，所以在单核处理器的情况下，性能效果并不会比Serial收集器效果更好。

**适用场景:**

服务端模式



> 在处理器核心非常多的环境中，可以使用`-XX:ParallelGCThreads`参数来限制垃圾收集的线程数。

```
-XX:SurivivorRatio
-XX:PretenureSizeThreshold
-XX:HandlePromotionFailure
-XX:+/-UseParNewGC	启用/禁用ParNew收集器
```

### Parallel Scavenge收集器

**目标：**

Parallel Scavenge收集器是一款新生代收集器，基于标记-复制算法实现的收集器，并行多线程。关注吞吐量。

**特性：**

- Parallel Scavenge收集器是一个多线程工作的收集器。
- 关注吞吐量，以达到一个控制的吞吐量。

> 吞吐量
>
> 吞吐量就是处理器用于运行用户代码的时间与处理器总消耗时间的比值。
>
> ```
> 吞吐量 = 运行用户代码时间/运行用户代码时间+运行垃圾收集时间
> ```
>
> 例如：虚拟机完成某个任务，用户代码加上垃圾收集总共耗费100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。

**适用场景:**

停顿时间越短就越适合需要与用户交互或需要保证服务响应质量的程序，良好的响应速度能提升用户体验。

高吞吐量以最高效率的利用处理器资源，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的分析任务。

**Parallel Scavenge收集器相关参数**

```
-XX:MaxGCPauseMillis
-XX:GCTimeRatio
-XX:+UseAdaptiveSizePolicy
```

- `-XX:MaxGCPauseMillis`

  该参数用于控制垃圾收集最大停顿时间。其值是一个大于0的毫秒数，收集器将尽力保证内存回收花费的时间不超过用户设定的值。需要注意的是，这个数值并不是越小垃圾收集速度就越快，垃圾收集停顿时间缩短是以牺牲吞吐量和新生代空间为代价换取的：系统把新生代调得小一些，收集300MB新生代肯定比收集500MB快，但这也直接导致垃圾收集发生得更频繁，原来10秒收集一次，每次停顿100毫秒，现在变成5秒收集一次，每次停顿70毫秒。停顿时间的确在下降，但吞吐量也降下来了。

- `-XX:GCTimeRatio`

  该参数用于控制吞吐量的大小。其值是一个大于0小于100的整数，也就是垃圾收集时间占总时间的比率，相当于吞吐量的倒数。譬如把此参数设置为19，那允许的最大垃圾收集时间占总时间的5%（即1/1+19），默认为99，即允许最大1%（即1/1+99）的垃圾收集时间。

- `-XX:+UseAdaptiveSizePolicy`

  该参数是一个开关参数，用于控制是否开启垃圾收集的自适应的调节策略（GC Ergonomics）。所谓自适应调节策略是指当此参数被激活之后，就不需要人工指定新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX:SurvivorRatio）、晋升老年代对象的大小（-XX:PretenureSizeThreshold）等细节参数，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大吞吐量。

### Serial Old收集器

**特性：**

- Serial Old收集器是Serial的老年代版本。
- Serial Old收集器是一个单线程工作的收集器。
- Serial Old收集器在进行垃圾收集时，必须暂停其他所有工作线程，直到它收集结束。
- Serial Old收集器作用于老年代。
- 使用标记-整理算法。

**原理：**

Serial Old收集器运行过程：

![](https://img2018.cnblogs.com/blog/1217276/201904/1217276-20190420020324547-2095827038.png)

**适用场景:**

主要作用于客户端模式。

如果在服务端模式下，有两种用途：

- 一种是在JDK5以及之前的版本中与Parallel Scavenge收集器搭配使用
- 另一种就是作为CMS收集器发生失败时的后备预案，在并发收集发生Concurrent Mode Failure时使用。

相关参数

```
-XX:+/-UseParallelGC
```

### Parallel Old收集器

特性：

- Parallel Old收集器是Parallel Scavenge收集器的老年代版本。
- Parallel Old收集器是一个多线程工作的收集器。
- 基于标记-整理算法实现。
- 作用于老年代。

**原理：**

Parallel Old收集器运行过程：

![](https://img2018.cnblogs.com/blog/1217276/201904/1217276-20190420105311115-504871477.png)

**适用场景:**

Parallel Old收集器是在JDK6才开始提供的，主要用于与Parallel Scavenge收集器配合使用。

**相关参数**

```java
-XX:+/-UseParallelOldGC	启用/禁用Parallel Old收集器
```

### CMS收集器

**目标**

CMS（Concurrent Mark Sweep）收集器是一种以获得最短回收停顿时间为目标的收集器。

> Parallel Scavenge收集器也可以控制停顿时间，但它主要关注的是吞吐量。

**特性**

- 基于标记-清除算法。
- 关注停顿时间。
- 默认启动回收线程数是(处理器核心数量+3)/4。

**原理**

CMS的垃圾收集过程分为四个步骤：

1. 初始标记（CMS initial mark ）
2. 并发标记（CMS concurrent mark）
3. 重新标记（CMS remark）
4. 并发清除（CMS concurrent sweep）



- 初始标记需要Stop The World，其仅仅只是标记一些GC Roots能直接关联到的对象，速度很快。
- 并发标记是从GC Roots的直接关联对象开始遍历整个对象图，这个过程耗时较长，但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行。
- 重新标记需要Stop The World，其是为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录。这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短。
- 并发清除，清理删除掉标记阶段判断已经死亡的对象，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的。

CMS收集器运行过程：

![](https://img2018.cnblogs.com/blog/1217276/201904/1217276-20190420113715103-1317893404.png)

**浮动垃圾问题**

> 浮动垃圾
>
> 在CMS的并发标记和并发清理阶段，用户线程是还在继续运行的，程序在运行，自然就还会伴随有新的垃圾对象不断产生，但这一部分垃圾对象是出现在标记过程结束以后，CMS无法在当此收集中处理掉它们，只好留待下一次垃圾收集时再清理掉。这一部分垃圾就称为浮动垃圾。

基于CMS收集器会产生浮动垃圾的原因，在垃圾收集时，就必须为用户线程预留足够的内存空间。因此CMS收集器不会像其他收集器那样等待老年代几乎被填满了再进行垃圾收集。

> CMS收集器为用户线程在老年代预留内存空间百分比：
>
> JDK5-默认使用68%触发。
>
> JDK6-默认使用92%触发。

可以通过参数`-XX:CMSInitiatingOccupancyFraction`设置CMS触发垃圾收集的百分比，数值越高就可以降低内存回收的频率，获得更好的性能。

带来的问题：预留的内存无法满足程序分配新对象的需要。

解决方式：当预留的内存无法满足程序分配新对象的需要时，CMS收集器就会出现一次“并发失败（Concurrent Mode Failure）”，虚拟机将启动后备预案：冻结用户线程的执行，临时启用Serial Old收集器来重新进行老年代的垃圾收集。

带来的影响：停顿时间更长。

**空间碎片问题**

CMS收集器基于标记-清除算法，而此会产生大量空间碎片。

解决方式：当不得不触发Full GC时，开启内存碎片整理。

带来的影响：停顿时间变长。

有如下两个参数可以影响内存碎片的整理：

1. `-XX:+/-UseCMS-CompactAtFullCollection`：用于在CMS收集器不得不进行Full GC时开启内存碎片整理。（默认开启。JDK9已被废弃）
2. `-XX:+/-CMSFullGCsBefore-Compaction`：要求CMS不每次Full GC时都执行内存碎片整理，而是在执行若干次不整理空间的Full GC之后，下一次进入Full GC前先进行内存碎片整理。（默认0，表示每次进入Full GC时都进行碎片整理。JDK9已被废弃）

**优点**

并发收集、低停顿

**缺点**

1. CMS收集器对处理器资源非常敏感。
2. CMS收集器无法处理“浮动垃圾（Floating Garbage）”，有可能出现“Con-current Mode Failure”失败进而导致另一次完全“Stop The World”的Full GC的产生。
3. CMS收集器基于标记-清除算法，将会产生大量空间碎片。空间碎片过多时，将会给大对象分配带来很大麻烦，往往会出现老年代还有很多剩余空间，但就是无法找到足够大的连续空间来分配当前对象，而不得不提前触发一次Full GC。

**相关参数**

```
-XX:CMSInitiatingOccupancyFraction	设置CMS收集器触发垃圾收集的老年代被使用空间的百分比
-XX:+/-UseCMS-CompactAtFullCollection	开启CMS收集器触发Full GC时内存空间碎片整理。
-XX:CMSFullGCsBefore-Compaction	设置CMS收集器几次不进行内存空间碎片整理的Full GC。
-XX:+/-UseConcMarkSweepGC	启用/禁用CMS收集器
```

### Garbage First收集器













![](https://img2018.cnblogs.com/blog/1217276/201904/1217276-20190420144927628-1735440321.png)












```shell
C:\Users\dandelion>java -help
用法: java [-options] class [args...]
           (执行类)
   或  java [-options] -jar jarfile [args...]
           (执行 jar 文件)
其中选项包括:
    -d32          使用 32 位数据模型 (如果可用)
    -d64          使用 64 位数据模型 (如果可用)
    -server       选择 "server" VM
                  默认 VM 是 server.

    -cp <目录和 zip/jar 文件的类搜索路径>
    -classpath <目录和 zip/jar 文件的类搜索路径>
                  用 ; 分隔的目录, JAR 档案
                  和 ZIP 档案列表, 用于搜索类文件。
    -D<名称>=<值>
                  设置系统属性
    -verbose:[class|gc|jni]
                  启用详细输出
    -version      输出产品版本并退出
    -version:<值>
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  需要指定的版本才能运行
    -showversion  输出产品版本并继续
    -jre-restrict-search | -no-jre-restrict-search
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  在版本搜索中包括/排除用户专用 JRE
    -? -help      输出此帮助消息
    -X            输出非标准选项的帮助
    -ea[:<packagename>...|:<classname>]
    -enableassertions[:<packagename>...|:<classname>]
                  按指定的粒度启用断言
    -da[:<packagename>...|:<classname>]
    -disableassertions[:<packagename>...|:<classname>]
                  禁用具有指定粒度的断言
    -esa | -enablesystemassertions
                  启用系统断言
    -dsa | -disablesystemassertions
                  禁用系统断言
    -agentlib:<libname>[=<选项>]
                  加载本机代理库 <libname>, 例如 -agentlib:hprof
                  另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help
    -agentpath:<pathname>[=<选项>]
                  按完整路径名加载本机代理库
    -javaagent:<jarpath>[=<选项>]
                  加载 Java 编程语言代理, 请参阅 java.lang.instrument
    -splash:<imagepath>
                  使用指定的图像显示启动屏幕
有关详细信息, 请参阅 http://www.oracle.com/technetwork/java/javase/documentation/index.html。
```





```shell
[root@localhost ~]# jps -help
usage: jps [--help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
    -? -h --help -help: Print this help message and exit.
```



```shell
[root@localhost ~]# jinfo -?
Usage:
    jinfo <option> <pid>
       (to connect to a running process)

where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
    <no option>          to print both VM flags and system properties
    -? | -h | --help | -help to print this help message
```





## jstat

```shell
C:\Users\dandelion>jstat -help
Usage: jstat --help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
  -? -h --help  Prints this help message.
  -help         Prints this help message.
```





```shell
[root@izbp12mpi6ej8nhsdjjk4kz ~]# jmap -?
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> doesnot
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```



| options                | descrition                                                   |
| ---------------------- | ------------------------------------------------------------ |
| `-heap`                | 显示java堆详细信息，例如使用哪种回收器，参数配置，分代状况等。只在Linux和Solaris平台有效。 |
| `-histo`               |                                                              |
| `-clatats`             |                                                              |
| `-finalizerinfo`       |                                                              |
| `-dump:<dump-options>` |                                                              |
| `-F`                   |                                                              |
| `-J<flag>`             |                                                              |
| `-h | -help`           |                                                              |



## Class file structure



### case analysis

```java
package com.example.jvm.clazz;
public class TestClass {
    private int m;
    public int inc(){
        return m + 1;
    }
}
```





```java
CA FE BA BE 00 00 00 36 00 13 0A 00 04 00 0F 09 
00 03 00 10 07 00 11 07 00 12 01 00 01 6D 01 00 
01 49 01 00 06 3C 69 6E 69 74 3E 01 00 03 28 29 
56 01 00 04 43 6F 64 65 01 00 0F 4C 69 6E 65 4E 
75 6D 62 65 72 54 61 62 6C 65 01 00 03 69 6E 63 
01 00 03 28 29 49 01 00 0A 53 6F 75 72 63 65 46 
69 6C 65 01 00 0E 54 65 73 74 43 6C 61 73 73 2E 
6A 61 76 61 0C 00 07 00 08 0C 00 05 00 06 01 00 
1F 63 6F 6D 2F 65 78 61 6D 70 6C 65 2F 6A 76 6D 
2F 63 6C 61 7A 7A 2F 54 65 73 74 43 6C 61 73 73 
01 00 10 6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 
65 63 74 00 21 00 03 00 04 00 00 00 01 00 02 00 
05 00 06 00 00 00 02 00 01 00 07 00 08 00 01 00 
09 00 00 00 1D 00 01 00 01 00 00 00 05 2A B7 00 
01 B1 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 
00 03 00 01 00 0B 00 0C 00 01 00 09 00 00 00 1F 
00 02 00 01 00 00 00 07 2A B4 00 02 04 60 AC 00 
00 00 01 00 0A 00 00 00 06 00 01 00 00 00 08 00 
01 00 0D 00 00 00 02 00 0E  
```

#### magic

`CA FE BA BE`

#### minor_version

`00 00` 小版本

#### magic_version

`00 36` 大版本

#### constant Pool

由于常量池中常量的数量是固定的，所以在常量池的入口需要放置一项u2可惜的数据，代表常量池容量计数值（constant_pool_count）。与Java中语言习惯不一样的是，这个容量计数是从1而不是从0开始，例如常量池容量为十六进制数0x13，即十进制19，这就代表常量池中有18向常量，索引值范围为1~18。设计者将第0项常量空出来是有特殊考虑的，这样做的目的在于满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义，这种情况就可以把索引值置位0来表示。

注意：Class文件结构只有常量池的容量计数是从1开始，对于其他集合类型，包括接口索引集合、字段表集合、方法表集合等的容量计数都与一般习惯相同，是从0开始的。

首先是常量池第0位

`00`

##### constant_pool_count

十六进制数0x13，即十进制数19，表示当前常量池中有18个常量。

##### constant_pool[constant_pool_count - 1]

1. **The one constant**

   tag值为0x0A，即十进制10，查看**常量池项目类型**表，可知为`CONSTANT_Methodref_info`表，共占用5个字节，如下：

   ```
   CONSTANT_Methodref_info {
       u1 tag;
       u2 class_index;
       u2 name_and_type_index;
   }
   ```

   十六进制数：0A 00 04 00 0F

2. **The twe constant**

   tag值为0x09，即十进制9，查看**常量池项目类型**表，可知为`CONSTANT_Fieldref_info`表，共占用5个字节，如下：

   ```
   CONSTANT_Fieldref_info {
       u1 tag;
       u2 class_index;
       u2 name_and_type_index;
   }
   ```

   十六进制数：09 00 03 00 10

3. **The third constant**

   tag值为0x07，即十进制7，查看**常量池项目类型**表，可知为`CONSTANT_Class_info`表，共占用3个字节，如下：

   ```
   CONSTANT_Class_info {
       u1 tag;
       u2 name_index;
   }
   ```

   十六进制数：07 00 11

4. **The four constant**

   07 00 12

5. **The five constant**

   tag值为0x01，即十进制1，查看**常量池项目类型**表，可知为`CONSTANT_Utf8_info`表，共占用1+2+length个字节，如下：

   ```
   CONSTANT_Utf8_info {
       u1 tag;
       u2 length;
       u1 bytes[length];
   }
   ```

   十六进制数：01 00 01 6D

01 00 01 => 6D => m

6. **The six constant**

   tag值为0x01，

第六个常量

01 00  01 => 49 => I

第七个常量

01 00 06 => 3C 69 6E 69 74 3E => <init>

第八个常量

01 00 03 => 28 29 56 => ()V

第九个常量

01 00 04 => 43 6F 64 65 => Code

第十个常量

01 00 0F => 4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65 => LineNumberTable

第十一个常量

01 00 03 =>69 6E 63 => inc

第十二个常量

01 00 03 =>28 29 49 => ()I

第十三个常量

01 00 0A=>53 6F 75 72 63 65 46 69 6C 65 => SourceFile

第十四个常量

01 00 0E=>54 65 73 74 43 6C 61 73 73 2E 6A 61 76 61=>TestClass.java

第十五个常量

0x0C = 12,`CONSTANT_NameAndType_info`

```
CONSTANT_NameAndType_info {
    u1 tag;
    u2 name_index;
    u2 descriptor_index;
}
```

0C 00 07 00 08

第十六个常量

0C 00 05 00 06

第十七个常量

01 00 1F=>63 6F 6D 2F 65 78 61 6D 70 6C 65 2F 6A 76 6D 2F 63 6C 61 7A 7A 2F 54 65 73 74 43 6C 61 73 73

=>com/example/jvm/clazz/TestClass

第十八个常量

01 00 10=>6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 65 63 74=>java/lang/Object



#### access_flags

紧接着是访问标志符

0x0021

#### this_class

0x0003

#### super_class

0x0004

#### interfaces_count

0x0000

#### interfaces[interfaces_count]



#### fields

继类索引，父类索引，接口索引之后是字段表集合，字段表集合如下：

```
field_info {
    u2 access_flags;
    u2 name_index;
    u2 descriptor_index;
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```

查看class format structure，在字段表集合之前还有两个字节，代表字段表的数量，所以对应的十六进制数如下：

```
00 01 00 02 00 05 00 06 00 00 00
```

字段表数量为十六进制数0x0001，即十进制1，即只有一个字段。

access_flags为十六进制数0x0002，查看field access flags table，可知访问标志位`private`。

name_index为十六进制数0x0005，即十进制5，对应常量表的第五个常量，查看第五个常量，为`CONSTANT_utf8_info`表，对应字段名称`m`。

descriptor_index为十六进制数0x0006，即十进制6，对应常量表第六个常量，查看第六个常量，为`CONSTANT_utf8_info`，对应字段类型`I`，再查看字段描述符标识字段含义，`I`为基本类型`int`。

attributes_count为十六进制数0x0000，即没有当前字段对应的属性表。

所以，可以得出改字段为：

```java
private int m;
```



##### fields_count

##### fields[fields_count]

#### methods

```
method_info {
    u2 access_flags;
    u2 name_index;
    u2 descriptor_index;
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```

00 02 => 00 01 00 07 00 08 00 01

首先两个字节表明是方法的数量

方法表数量为十六进制0x0002，即十进制2，代表有两个方法。

第一个方法：

access_flags为十六进制数0x0001，查看method access flags，访问标记为`public`。

name_index为十六进制数0x0007，即十进制7，对应常量表第7个常量，查看第7个常量，为`CONSTANT_utf8
_info`表，对应方法名称`<init>`，即时构造方法。

descriptor_index为十六进制数0x0008，即十进制8，对应常量表第8个常量，查看第8个常量，为`CONSTANT_utf8_info`表，对应的方法返回值 `()V`。

attributes_count为十六进制数0x0001，



<<<<<<< HEAD
00 
09 00 00 00 1D 00 01 00 01 00 00 00 05 2A B7 00 
01 B1 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 
00 03



00 01 00 0B 00 0C 00 01 00 09 00 00 00 1F 
00 02 00 01 00 00 00 07 2A B4 00 02 04 60 AC 00 
00 00 01 00 0A 00 00 00 06 00 01 00 00 00 08
=======


```
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

00 09 00 00 00 1D 00

attribute_name_index为十六进制0x0009，即十进制9，对应常量表第9个常量，查看第9个常量，为`CONSTANT_utf8_info`表，对应属性名Code。

attribute_length为十六进制0x0000001D，即十进制29，即有29个属性，每个属性站一个字节，即29个字节

info：

```java
14 -                00 01 00 01 00 00 00 05 2A B7 00 
15 - 01 B1 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 
16 - 00 03
```



第一个方法Code属性

- attribute_name_index：00 01
- attribute_length： 
- max_stack：00 01 
- max_locals：00 00 00 05 = 2A B7 00 01 B1
  - 2A => aload_0 => 将第一个引用类型本地变量推送至栈顶
  - B7 => invokespecial => 调用超类的构造方法，实例初始化方法，私有方法
  - 01 => nop => 什么都不做 
  - B1 => return => 从当前方法返回void
- code_length：00 00 00 01
- code：
- exception_table_length：
- ：
- ：
- attributes_count：
- attribute_info：

```
Code_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 max_stack;
    u2 max_locals;
    u4 code_length;
    u1 code[code_length];
    u2 exception_table_length;
    { 	u2 start_pc;
        u2 end_pc;
        u2 handler_pc;
        u2 catch_type;
    } exception_table[exception_table_length];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```



第二个方法：

```java
16 -       00 01 00 0B 00 0C 00 01 00 09 00 00 00 1F 
17 - 00 02 00 01 00 00 00 07 2A B4 00 02 04 60 AC 00 
18 - 00 00 01 00 0A 00 00 00 06 00 01 00 00 00 08
```

- access_flags : 00 01 = 1 = ACC_PUBLIC

- name_index : 00 0B = 11 = inc

- descriptor_index : 00 0C = 12 =  ()I => ()表示无参，l表示返回值int类型

- attributes_count : 00 01 = 1 = 1个属性

- attribute_info ：

  - attribute_name_index：00 09 = 9 = Code

  - attribute_length：00 00 00 1F = 31

  - info：

    ```java
    17 - 00 02 00 01 00 00 00 07 2A B4 00 02 04 60 AC 00 
    18 - 00 00 01 00 0A 00 00 00 06 00 01 00 00 00 08
    ```





第二个方法Code属性

```java
17 - 00 02^00 01^00 00 00 07^2A B4 00 02 04 60 AC^00 
18 - 00 00 01 00 0A 00 00 00 06 00 01 00 00 00 08
```

- attribute_name_index：00 02
- attribute_length：
- max_stack：00 01 
- max_locals：00 00 00 07 = 2A B4 00 02 04 60^AC
  - aload_0 => 将第一个引用类型本地变量推送至栈顶
  - getfield => 获取指定类的实例域，并将其值压入栈顶
  - iconst_m1 => 将int型_上推送至栈顶
  - iconst_1 =>  将int型1推送至栈顶
  - iadd => 将栈顶两int型数值相加并将结果压入栈顶
  - ireturn => 从当前方法返回int
- code_length：00 00 00 01
- code：
- exception_table_length：
- ：
- ：
- attributes_count：
- attribute_info：



```
Code_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 max_stack;
    u2 max_locals;
    u4 code_length;
    u1 code[code_length];
    u2 exception_table_length;
    { 	u2 start_pc;
        u2 end_pc;
        u2 handler_pc;
        u2 catch_type;
    } exception_table[exception_table_length];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```




>>>>>>> 2a07e6ce0269c13e6a2f420b105ff9e5fddf674b

##### methods_count



##### methods[methods_count]



```java
1  - CA FE BA BE 00 00 00 36 00 13 0A 00 04 00 0F 09 
2  - 00 03 00 10 07 00 11 07 00 12 01 00 01 6D 01 00 
3  - 01 49 01 00 06 3C 69 6E 69 74 3E 01 00 03 28 29 
4  - 56 01 00 04 43 6F 64 65 01 00 0F 4C 69 6E 65 4E 
5  - 75 6D 62 65 72 54 61 62 6C 65 01 00 03 69 6E 63 
6  - 01 00 03 28 29 49 01 00 0A 53 6F 75 72 63 65 46 
7  - 69 6C 65 01 00 0E 54 65 73 74 43 6C 61 73 73 2E 
8  - 6A 61 76 61 0C 00 07 00 08 0C 00 05 00 06 01 00 
9  - 1F 63 6F 6D 2F 65 78 61 6D 70 6C 65 2F 6A 76 6D 
10 - 2F 63 6C 61 7A 7A 2F 54 65 73 74 43 6C 61 73 73 
11 - 01 00 10 6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 
12 - 65 63 74 00 21 00 03 00 04 00 00 00 01 00 02 00 
13 - 05 00 06 00 00@00 02 00 01 00 07 00 08 00 01 00 
14 - 09 00 00 00 1D^00 01 00 01 00 00 00 05 2A B7 00 
15 - 01 B1 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 
16 - 00 03@00 01 00 0B 00 0C 00 01 00 09 00 00 00 1F 
17 - 00 02 00 01 00 00 00 07 2A B4 00 02 04 60 AC 00 
18 - 00 00 01 00 0A 00 00 00 06 00 01 00 00 00 08 00 
19 - 01 00 0D 00 00 00 02 00 0E  
```
<<<<<<< HEAD
CA FE BA BE 00 00 00 36 00 13 0A 00 04 00 0F 09 
00 03 00 10 07 00 11 07 00 12 01 00 01 6D 01 00 
01 49 01 00 06 3C 69 6E 69 74 3E 01 00 03 28 29 
56 01 00 04 43 6F 64 65 01 00 0F 4C 69 6E 65 4E 
75 6D 62 65 72 54 61 62 6C 65 01 00 03 69 6E 63 
01 00 03 28 29 49 01 00 0A 53 6F 75 72 63 65 46 
69 6C 65 01 00 0E 54 65 73 74 43 6C 61 73 73 2E 
6A 61 76 61 0C 00 07 00 08 0C 00 05 00 06 01 00 
1F 63 6F 6D 2F 65 78 61 6D 70 6C 65 2F 6A 76 6D 
2F 63 6C 61 7A 7A 2F 54 65 73 74 43 6C 61 73 73 
01 00 10 6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 
65 63 74 00 21 00 03 00 04 00 00 00 01 00 02 00 
05 00 06 00 00@00 02 00 01 00 07 00 08 00 01 00 
09 00 00 00 1D 00 01 00 01 00 00 00 05 2A B7 00 
01 B1 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 
00 03@00 01 00 0B 00 0C 00 01 00 09 00 00 00 1F 
00 02 00 01 00 00 00 07 2A B4 00 02 04 60 AC 00 
00 00 01 00 0A 00 00 00 06 00 01 00 00 00 08 00 
01 00 0D 00 00 00 02 00 0E  
=======





```java
C:\Users\dandelion\Desktop>javap -verbose TestClass.class
Classfile /C:/Users/dandelion/Desktop/TestClass.class
  Last modified 2019年3月28日; size 297 bytes
  MD5 checksum 28c5755e84efa06bef94473ec2353bf0
  Compiled from "TestClass.java"
public class com.example.jvm.clazz.TestClass
  minor version: 0
  major version: 55
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #3                          // com/example/jvm/clazz/TestClass
  super_class: #4                         // java/lang/Object
  interfaces: 0, fields: 1, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         // com/example/jvm/clazz/TestClass.m:I
   #3 = Class              #17            // com/example/jvm/clazz/TestClass
   #4 = Class              #18            // java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               TestClass.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = NameAndType        #5:#6          // m:I
  #17 = Utf8               com/example/jvm/clazz/TestClass
  #18 = Utf8               java/lang/Object
{
  public com.example.jvm.clazz.TestClass();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 2: 0

  public int inc();
    descriptor: ()I
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field m:I
         4: iconst_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 5: 0
}
SourceFile: "TestClass.java"
>>>>>>> 2a07e6ce0269c13e6a2f420b105ff9e5fddf674b
```







### 常量池的项目类型

| type                               | tag  | description              |
| ---------------------------------- | ---- | ------------------------ |
| `CONSTANT_Utf8_info`               | 1    | UTF-8编码的字符串        |
| `CONSTANT_Integer_info`            | 3    | 整型字面量               |
| `CONSTANT_Float_info`              | 4    | 浮点型字面量             |
| `CONSTANT_Long_info`               | 5    | 长整型字面量             |
| `CONSTANT_Double_info`             | 6    | 双精度浮点型字面量       |
| `CONSTANT_Class_info`              | 7    | 类或接口的符号引用       |
| `CONSTANT_String_info`             | 8    | 字符串类型字面量         |
| `CONSTANT_Fieldref_info`           | 9    | 字段的符号引用           |
| `CONSTANT_Methodref_info`          | 10   | 类中方法的符号引用       |
| `CONSTANT_InterfaceMethodred_info` | 11   | 接口中方法的引用符号     |
| `CONSTANT_NameAndType_info`        | 12   | 字段或方法的部分引用符号 |
| `CONSTANT_MethodHandle_info`       | 15   | 表示方法句柄             |
| `CONSTANT_MethodType_info`         | 16   | 标识方法类型             |
| `CONSTANT_InvokeDenamic_info`      | 18   | 表示一个动态方法的调用点 |



#### `CONSTANT_Utf8_info`





#### `CONSTANT_Fieldref_info`

```
CONSTANT_Fieldref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```

#### `CONSTANT_Methodref_info`

```
CONSTANT_Methodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```

#### `CONSTANT_InterfaceMethodref_info`

```
CONSTANT_InterfaceMethodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```



### Access flags table

| flag name      | flag value | 含义                                                         |
| -------------- | ---------- | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001     | 是否为public类型                                             |
| ACC_FINAL      | 0x0010     | 是否被什么为final，只有类可设置                              |
| ACC_SUPER      | 0x0020     | 是否允许使用invokespecial字节码指令的新语意，invokespecial指令的语意在JDK 1.0.2发生过改变，为了区别这条指令使用哪种语意，JDK 1.0.2之后编译出来的类的这个标志都必须为真。 |
| ACC_INTERFACE  | 0x0200     | 标识这是一个接口                                             |
| ACC_ABSTRAT    | 0x0400     | 是否为abstract类型，对于接口或者抽象类来说，此标志值为真，其他类值为假 |
| ACC_SYNTHETIC  | 0x1000     | 标识这个类并非由用户代码产生的                               |
| ACC_ANNOTATION | 0x2000     | 标识这时一个注解                                             |
| ACC_ENUM       | 0x4000     | 标识这是一个枚举                                             |

### Field access flags table

| flag name     | flag value | 含义                     |
| ------------- | ---------- | ------------------------ |
| ACC_PUBLIC    | 0x0001     | 字段是否public           |
| ACC_PRIVATE   | 0x0002     | 字段是否private          |
| ACC_PROTECTED | 0x0004     | 字段是否protected        |
| ACC_STATIC    | 0x0008     | 字段是否static           |
| ACC_FINAL     | 0x0010     | 字段是否final            |
| ACC_VOLATILE  | 0x0040     | 字段是否volatile         |
| ACC_TRANSIENT | 0x0080     | 字段是否transient        |
| ACC_SYNTHETIC | 0x1000     | 字段是否由编译器自动产生 |
| ACC_ENUM      | 0x4000     | 字段是否enum             |



### 描述符标识字段含义

| 标识字段 | 含义           | 标识字段 | 含义                          |
| -------- | -------------- | -------- | ----------------------------- |
| B        | 基本类型byte   | J        | 基本类型long                  |
| C        | 基本类型char   | S        | 基本类型short                 |
| D        | 基本类型double | Z        | 基本类型boolean               |
| F        | 基本类型float  | V        | 特殊类型void                  |
| I        | 基本类型int    | L        | 对象类型，如Ljava/lang/Object |

### Method access flags

| flag name        | flag value | 含义                           |
| ---------------- | ---------- | ------------------------------ |
| ACC_PUBLIC       | 0x0001     | 方法是否为public               |
| ACC_PRIVATE      | 0x0002     | 方法是否为private              |
| ACC_PROTEDTED    | 0x0004     | 方法是否为protected            |
| ACC_STATIC       | 0x0008     | 方法是否为static               |
| ACC_FINAL        | 0x0010     | 方法是否为final                |
| ACC_SYNCHRONIZED | 0x0020     | 方法是否为synchronized         |
| ACC_BRIDGE       | 0x0040     | 方法是否由编译器产生的桥接方法 |
| ACC_VARARGS      | 0x0080     | 方法是否接受不定参数           |
| ACC_NATIVE       | 0x0100     | 方法是否为native               |
| ACC_ABSTRACT     | 0x0400     | 方法是否为abstract             |
| ACC_STRICTFP     | 0x0800     | 方法是否为strictfp             |
| ACC_SYNTHETLC    | 0x1000     | 方法是否由编译器自动产生       |

