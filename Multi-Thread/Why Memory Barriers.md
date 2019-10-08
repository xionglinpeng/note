## Why Memory Barriers

## 一、前言

到底是什么原因导致CPU的设计者把memory barrier这样的大招强加给可怜的，不知情的SMP软件设计者？

一言蔽之，性能，因为对内存访问顺序的重排序可以获得更好的性能，如果某些场合下，程序的逻辑正确性需要内存访问顺序与程序顺序一致，例如：同步原语，那么SMP软件工程师可以使用memory barrier这样的工具阻止CPU对内存访问顺序的优化。

如果你想了解更多，需要充分理解CPU缓存是如何工作的，以及如何让CPU缓存更好的工作，本文的主要内容包括：

1. 描述缓存（cache）结构。
2. 描述缓存一致性协议（cache-coherency protocol）如何保证缓存（cache）一致性。
3. 描述存储缓冲（store buffers）和失效队列（invalidate queues）如何获取更好的性能。

## 二、Cache Structure

现在CPU的速度要远快于内存系统（memory system）。一个2006年的CPU可以每ns执行10条指令，但是却需要几十ns来从主内存（man memory）中获取数据。这个速度的差异（超过2个数量级）使得现代CPU一般会有几兆（MB）的cache。当然这些缓存可以分成若干级别（level），最靠近CPU那个level的cache可以在一个cycle内完成对内存的访问。我们抽象现代计算机系统的cache结构如下：

![](http://www.wowotech.net/content/uploadfile/201512/f67b56f268b224252770556a803050ba20151210110912.gif)

CPU cache和memory system使用固定大小的数据块来进行交互，这个数据块被称为缓存行（cache line），cache line的size一般是2的整数次幂，根据设计的不同，从16B到256B不等。当CPU首次访问某个数据的时候，它没有在CPU cache中，我们称之为==cache miss==（更准确的说法是startup或者warmup cache line）。在这种情况下，CPU需要花费几百个cycle去把该数据对应的cache line从memory中加载到CPU cache中，而在这个过程中，CPU只能是等待那个耗时的内存操作完成。一旦完成了CPU cache数据的加载，随后的访问会由于数据在cache中而使得CPU全速运行。

运行一段时间之后，CPU cache的所有cache line都会被填充有效的数据，这时候，要加载新的数据到cache中必须将其他原来有效的cache数据“强制驱离”（一般选择最近最少使用的那些cache line）。这种cache miss被称为==capacity miss==，因为CPU cache的容量有限，必须为新数据找到空闲的cache line。有的时候，即便cache中还有空闲（idle）的cache line，旧的cache数据也会被“强制驱离”，以便为新的数据加载到cache中做准备。当然，这是和cache的组织有关。size比较大的cache往往实现成hash table（为了硬件性能），所有的cache line被分成了若干固定大小的hash buckets（更专业的术语叫set），这些hash buckets之间不是形成链表，而是类似阵列，具体如下图所示：

![](http://www.wowotech.net/content/uploadfile/201512/a85cd9e524856053cce8bef6185b722620151210110914.gif)

图中的cache一共有32个cache line，被组织成16个set，每个set有两个可选cache line，分别称之为way 0和way 1。每个cache line有256个Byte，在cache和memory交互cache line的时候，要求cache line中数据地址对其在256个字节上。256B的cache line size稍显大了一点，主要是为了16进制的算法简单一些，实际中level 0的CPU cache一般没有这么大。如果用专业术语来说的话，上面的这种cache被称为twe way set-associative cache。这种cache的组织类似软件中有16个buckets的hash table，每个buckets（注意，这里是复数）中有两个butcket，最多可以放两个数据元素。total cache size以及associativity（2 way）被称为cache的几何结构。由于硬件实现，因此hash function（选择哪一个buckets）非常简单：从内存地址中选择4个bit即可。

在上图中，每一个cell表示一个cache line，保存256B数据，空的cell表示该cache line没有数据，是idle状态的，缓存数据的cache line标记了其保存数据对应memory address。由于有256B对其要求，因此地址的低8位都是0，而8~11这四个bit用来选择set。

当程序顺序访问了0x12345000到0x12345EFF之间的数据的时候，cache中的前15个set的way 0 cache line被加载了数据。随后对0x43210E00到0x43210EFF数据的访问，导致cache的第15个set的way 1 cache line也被加载了数据。OK，上图中的cache的状态就是这样的。这时候，我们一起看看后续cache的操作情况。如果程序访问0x1233000地址的数据，那么set 0被选中，由于way 0已经是保存了数据，因此way 1被用来缓存本次数据访问的内容。如果程序访问0x12345F00地址的数据，那么set 15被选中，由于way 0和way 1都是idle的，因此way 0被用来缓存本次数据访问的内容。但是如果访问0x1233E00这个地址开始的256B数据块的时候，问题来了，这时候，set 14（0xE）已经满了，way 0和way 1这两个cache line都已经加载了数据，怎么办？当然是把其中之一赶出去，为新来的数据让出地方。如果被赶出去的数据随后又被访问，这时候的cache miss被称为==associativity miss==。

到目前为止，我们只考虑了CPU读取数据的情况，如果写入数据会怎样呢？在某个CPU写入数据之前，有一点很重要，即所有的CPU需要对该数据的内容达成共识。因此，A CPU写入之前，需要先将其他CPU cache中的数据设定为无效。只有这个操作完成之后，A CPU才能安全的写入数据，而不会造成一致性的问题。如果该数据已经在A CPU的cache中，但是是read only的，这时候，该CPU不能直接操作cache line中的对应的数据（因为是read only的），这种cache miss被叫做==write miss==。一旦A CPU完成了invalidate其他CPU cache中的数据，该CPU可以不断的写或者读取其cache中的数据。（注意：为了表述方便，我这里给指定CPU命令为A）

稍后，如果其他CPU也要访问该数据，由于其他CPU的cache数据已经被设置为无效，因此，其他CPU的访问会导致cache miss。之所以如此，是因为前面A CPU在写入数据的时候，将其他CPU的cache数据设置为无效，这种cache miss被称为==communication miss==。之所以称为communication miss，是因为这种cache miss的发生是由于多个CPU使用共享内存进行通信（例如：互斥算法中的lock）。

毫无疑问，系统中的各个CPU在进行数据访问的时候有自己的视角（通过自己的CPU cache），因此小心的维持数据的一致性变得非常重要。如果不仔细进行设计，有可能在各个CPU对自己特定的CPU cache进行加载cache line、设置cache line无效、将数据写入cache line等动作中，把事情搞砸，例如数据丢失，或者更糟糕一些，不同的CPU在各自cache中看到不同的值。这些问题可以通过缓存一致性协议来保证，也就是下一节的内容。

## 三、Cache Coherency Protocols

缓存一致性协议用来管理cache line的状态，从而避免数据丢失或者数据一致性问题。这些协议可能非常复杂，定义几十个状态，本节我们只关心==MESI缓存一致性协议==的四个状态。

### 1、MESI状态

MESI是“modified”，“exclusive”，“shard”，“invalid”首字母的大写，当使用MESI缓存一致性协议的时候，cache line可以处于这四个状态中的一个，因此，HW工程师设计cache的时候，除了物理地址和具体的数据之外，还需要为==每一个cache line设计一个2-bit的tag来标识该cache line的状态==。

处于==modified==状态的cache line说明近期有过来自对应CPU的写操作，同时也说明该数据不会存在其他CPU对应的cache中。因此，处理modified状态的cache line也可以说是被该CPU独占。而又因为只有该CPU的cache保存了最新的数据（最终的memory中都没有更新），所以，该cache需要对该数据负责到底。例如根据请求，该cache将数据及其控制权传递到其他cache中，或者cache需要负责将数据写回到memory中，而这些操作都需要在再次使用该cache line之前完成。

==exclusive==状态和modified状态非常类似，唯一的区别是对应CPU还没有修改cache line中的数据，也正因为还没有修改数据，因此memory中对应的数据也是最新的。在exclusive状态下，CPU也可以不通知其他CPU cache，而直接对cache line进行操作，因为，exclusive状态也可以被认为是被该CPU独占，由于memory中的数据和cache line中的数据都是最新的，因此，CPU不需对exclusive状态的cache line执行写回操作或者将数据以及归属权转交给其他CPU cache，而直接再次使用该cache line（将cache line中的数据丢弃，用作他用）。

处于==share==状态的cache line，其数据可能在一个或者多个CPU cache中，因此处于这种状态的cache line，CPU不能直接修改cache line中的数据，而是需要首先和其他CPU cache进行沟通。和exclusive状态类似，处于share状态的cache line对应的memory中的数据也是最新的，因此，CPU可以直接丢弃cache line中的数据，而不必将其转交给其他CPU cache或者写回到memory中。

处于==invalid==状态的cache line是空的，没有数据。当新的数据要进入cache的时候，优选状态是invalid的cache line，之所以如此是因为如果选中其他状态的cache line，则说明需要替换cache line中的数据，而未来如果再次访问这个被替换的cache line中的数据的时候将遇到开销非常大的cache miss。

由于所有的CPU需要通过其cache看到一致性的数据，因此缓存一致性协议被用来协调cache line数据在系统中的移动。

### 2、MESI Protocol Message





























































