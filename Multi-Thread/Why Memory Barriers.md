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

在上节中描述的各种状态的迁移需要CPU之间的通信，如果所有CPU都是在一个共享的总线上的时候，下面的message就足够了：

1. Read："read"消息用来获取指定物理地址上的cache line数据。
2. Read Response：该消息携带了read消息请求的数据。read response可能来自memory，也可能来自其他的cache。例如：如果一个cache有read消息请求的数据并且该cache line的状态是modified，那么该cache必须以read response返回这个read消息，因为该cache中保存了最新的数据。
3. Invalidate：该命令用来将其他CPU缓存中的数据设定为无效。该命令携带物理地址的参数，其他CPU缓存在收到该命令后，必须进行匹配，发现自己的cache line中有该物理地址的数据，那么就将其移出并用Invalidate Acknowledge回应。
4. Invalidate Acknowledge：收到Invalidate消息的CPU缓存，在移出了其cache line中特定数据之后，必须发生Invalidate Acknowledge消息。
5. Read Invalidate：该消息中也包括了物理地址这个参数，以便说明其想要读取哪一个cache line中的数据。此外，该消息还同时有Invalidate消息的功效，即其他的cache在收到该命令之后，移除自己cache line中的数据。因此Read Invalidate消息实际上就是Read + Invalidate。发送Read Invalidate之后，cache期望收到一个Read Response以及多个Invalidate Acknowledge。
6. Writeback：该消息包含两个参数，一个是地址，另外一个是写回的数据。该消息用在modified状态的cache line被驱逐出境（给其他数据腾出地方）的时候发出，该命令用来将最新的数据写回到memory（或者其他CPU缓存中）。

有意思的是基于共享内存的多核系统其底层是基于消息传递的计算机系统。这也就意味着由多个SMP机器组成的共享内存的cluster系统在两个不同的level上使用了消息传递机制，一个是SMP内部消息传递，另外一个是SMP机器之间的。

### 3、MESI State Diagram

根据协议消息的发生和接收情况，cache line会在“modified”，“exclusive”，“shard”和“invalid”这四个状态之间迁移，具体如下图所示：

![](http://www.wowotech.net/content/uploadfile/201512/3920a2eecee145d71aa06155a3640efd20151210110953.gif)

对上图中的状态迁移解释如下：

Transition(a)：cache可以通过writeback事务将一个cache line的数据写回到memory中（或者下一级cache中），这时候，该cache line的状态从Modified迁移到Exclusive状态。对于CPU而言，cache line中的数据仍然是最新的，而且是该CPU独占的，因此可以不通知其他CPU缓存而直接修改。

Transition(b)：在Exclusive状态下，CPU可以直接将数据写入cache line，不需要其他操作。相应的，该cache line状态从Exclusive状态迁移到Modified状态。这个状态迁移过程不涉及BUS上的事务（即无需MESI协议消息的交互）。

Transition(c)：CPU在总线上收到一个Read Invalidate的请求，同时，该请求是针对一个处于Modified状态的cache line，在这种情况下，CPU必须将该cache line状态设置为无效，并且用Read Response和Invalida Acknowledge来回应收到的Read Invalidate的请求，完成整个总线事务。一个完成这个事务，数据被送往其他CPU缓存中，本地的copy已经不存在了。

Transition(d)：CPU需要执行一个原子的read-modify-write操作，并且其cache中没有缓存数据，这时候，CPU就会在总线上发送一个Read Invalidate用来请求数据，同时想独自霸占对该数据的所有权。该CPU的cache可以通过Read Response获取数据并加载cache line，通过，为了确保其独占的权利，必须收集所有其他CPU发来的Invalidate Acknowledge之后（其他CPU没有本底拷贝），完成整个总线事务。

Transition(e)：CPU需要执行一个原子的read-modify-write操作，并且其本地缓存中只读的缓存数据（cache line处于Shard状态），这时候，CPU就会在总线上发送一个Invalidate请求其他CPU清空自己的本地拷贝，以便完成其独自霸占对该数据的所有权的梦想。同样的，该CPU必须收集所有其他CPU发来的Invalidate Acknowledge之后，才算完成整个总线事务。

Transition(f)：在本CPU独自享受独占数据的时候，其他CPU发生Read请求，希望获取数据，这时候，本CPU必须以其本地cache line的数据回应，并以Read Reponse回应之前总线上的Read请求。这时候，本CPU失去了独占权，该cache line状态从Modified状态变成Shard状态（有可能也会进行写回的动作——写回到主内存）。

Transition(g)：这个迁移和f类似，只不过开始cache line的状态是Exclusive，cache line和memory的数据都是最新的，不存在写回的问题。总线上的操作也是在收到Read请求之后，以Read Response回应。

Transition(h)：如果CPU认为自己很快就会启动对处于Shard状态的caceh line的write操作，因此想提前霸占上改数据。因此，该CPU会发送Invalidate敦促其他CPU清空自己的本地拷贝，当收到全部其他CPU的Invalida Acknowledge之后，事务完成，本CPU上对应的cache line从Shard状态切换为Exclusive状态。还有另外一种方法也可以完成这个状态的切换：当所有其他的CPU对其本地拷贝的cache line进行写回操作，通过将cache line中的数据设为无效（主要是为了为新的数据腾些地方），这时候，本CPU坐享其成，直接获得了对该数据的独占权。

Transition(i)：其他的CPU进行一个原子的read-modify-write操作，但是，数据在本CPU的cache line中，因此，其他的那个CPU会发生Read Invalidate，请求该数据以及独占权。本CPU回送Read Reponse和Invalidate Acknowledge，一方面把数据转移到其他CPU的cache中，另一方面，清空自己的cache line。

Transition(j)：CPU想要进行write操作，但是数据不再本地缓存中，因此，该CPU首先发送Read Invalidate启动一次总线事务。在收到Read Response回应拿到数据，并收集到所有其他CPU发来的Invalida Acknowledge之后（确保其他CPU），完成整个总线事务，这时候的状态已经是Exclusive状态了。当write操作完成之后，该cache line的状态会从Exclusive状态迁移到Modified状态。

Transition(k)：本CPU执行读操作，发现本地缓存中没有数据，因此通过Read发生一次总线事务，来自其他的CPU本地缓存或者memory会通过Read Response回应，从而将该cache line从Invalid状态迁移到Shard状态。

Transition(l)：当cache line处于Shard状态的时候，说明在多个CPU的本地缓存中存在副本，因此，这些cache line中的数据都是只读的，一旦其他一个CPU想要执行数据写入的动作，必须先通过Invalidate获取该数据的独占权，而其他CPU会以Invalidate Acknowledge回应，清空数据并将其cache line从shard状态修改成Invalid状态。

### 4、MESI Protocol Example

OK，在理解了各种cache line状态，各种MESI协议消息以及状态迁移的描述之后，我们从cache line数据的角度来看看MESI协议是如何运作的。开始，数据保存在memory的0地址中，随后，该数据会穿行在四个CPU的本地缓存中。为了方便起见，我们让CPU本地缓存使用使用最简单的直接映射（Direct-mapped）的组织形式。具体的过程可以参考下面的图片：

![](http://www.wowotech.net/content/uploadfile/201512/76c88817a500ce79866c31354f00959420151210111005.gif)

第一列是操作序列号，第二列是执行操作的CPU，第三列是具体执行哪一种操作，第四列描述了各个CPU本地缓存中的cache line状态（用memory address/状态表示），最后一列描述了内存在0地址和8地址的的数据内容的状态：V表示最新的，和缓存一致，I表示不是最新的内容，最新的内容保存在cache中。

sequence 0：最开始的时候（sequence 0），各个CPU缓存中的cache line都是Invalid状态，而Memory中的数据都保存了最新的数据。随后（sequence 1），CPU 0执行了load操作，将address 0的数据加载到寄存器，这个操作使得保存在0地址的数据的那个cache line从Invalid状态迁移到shard状态。

sequence 2：随后（sequence 2），CPU 3也对address 0执行了load操作，导致其本地缓存上对应的cache line也切换到了shard状态。当然，这时候，memory仍然是最新的。

sequence 3：在sequence 3中，CPU 0执行了对address 8的load操作，由于address 0和address 8都是选择同一个cache set，而且，我们之前已经说过，该cache是直接映射（direct-mapped）的（即每个set都只有一个cache line），因此需要首先清空该cache line中的数据（该操作被称为Invalidation），由于cache line的状态是shard，因此不需要通知其他CPU。Invalidation本地缓存上的cache line之后，CPU 0的load操作将该cache line状态修改为shard状态（保存address 8的数据）。

sequence 4：CPU 2也开始执行load操作了（sequence 4），虽然是load操作，但是CPU知道程序随后会修改该值（不是原子操作read-modify-write，否则就是迁移到Modified状态了，也不是单纯的load操作，否则会迁移到Shard状态），因此向BUS发送可Read Invalidate命令，一方面获取该数据（自己的本地缓存中没有address 0的数据），另外，CPU 2向要独占该数据（因为随后要write）。这个操作导致CPU 3的cache line状态迁移到Invalid。当然，这时候，memory仍然是最新的有效数据。

sequence 5：CPU 2的Store操作很快到来（sequence 5），由于准备工作做的比较充分（Exclusive状态，独占该数据），CPU直接修改cache line中的数据（对应address 0），从而将其状态迁移到Modified状态，同时要注意的是：memory中的数据已经失效，不是最新的数据了，任何其他CPU发起对address 0的load操作都不能从memory中读取，而是通过嗅探（snoop）的方式从CPU 2的本地缓存中获取。

sequence 6：在sequence 6中，CPU 1对address 0的数据执行原子的加1操作，这时候CPU 1会发出Read Invalidate命令，将address 0的数据从CPU 2的缓存 line中嗅探得到，同时通过Invalidate其他CPU本地缓存的内容而获得独占的数据访问权。这时候CPU 2中的cahce line状态变成Invalid状态，而CPU 1将从Invalid状态迁移到Modified状态。

sequence 7：最后（sequence 7），CPU 1对address 8进行load操作，由于cache line被address 0占据，因此需要首先将其驱逐出cache，于是执行write back操作将address 0的数据写回到memory，同时发送Read命令，从CPU 0的缓存中获得数据加载其cache line，最后CPU 1的缓存变成了Shard状态（保存address 8的数据）。由于执行了write back操作，memory的address 0的数据又变成最新的有效数据了。

## 四、Stores Result in Unnecessary Stalls

在上面的现代计算机cache结构图，我们可以看出，针对某些特定地址的数据（在一个cache line中）重复的进行读写，这种结构可以获得很好的性能，不过，对于第一次写，其性能非常差。下面的这个图可以展示为何写性能差：

![](http://www.wowotech.net/content/uploadfile/201512/97abe4d93e6f06397ac1ccd459ca75e920151210111037.gif)

CPU 0发起一次对某个地址的写操作，但是本地缓存没有数据，该数据在CPU 1的本地缓存中，因此，为了完成写操作，CPU 0发出Invalidate命令，Invalidate其他CPU的缓存数据。只有完成了这些总线上的事务之后，CPU 0才能真正发起写的操作，这是一个漫长的等待过程。

但是，其实没必要等待这么长的时间，毕竟，物理CPU 1中的cache line保存有什么样子的数据，其实都没有意义，这个值都会被CPU 0新写入的值覆盖。

### 1、Store Buffers

有一种可以阻止CPU进入无聊等待状态的方法就是在CPU和缓存之间增加store buffer这个HW block，如下图所示：

![](http://www.wowotech.net/content/uploadfile/201512/ba5899824fccba75e192a435fc0e34bf20151210111047.gif)

一旦增加了store buffer，那么CPU 0就无需等待其他CPU的响应操作，只需要将要修改的内容放入store buffer中，然后继续执行就OK了。当cache line完成了总线事务，并更新了cache line的状态后，要修改的数据将从store buffer进入cache line。

这些store buffer对于CPU而言是本地的，如果系统是硬件多线程，那么每一个CPU核心拥有自己私有的store buffer，一个CPU只能访问自己私有的那个store buffer。在上图中，CPU 0不能访问CPU 1的store buffer，反之亦然。之所以做这样的限制是为了模块划分（各个CPU核心模块关心自己的事情，让缓存系统维护自己的操作），让硬件设计变得简单一些。store buffer增加了CPU连续写的性能，同时把各个CPU之间的通信的任务交给维护缓存一致性的协议。即便给每个CPU分配私有的store buffer，仍然引入了一下复杂性，我们将会在下面的两个小节中描述。

### 2、Store Forwarding

上文提到store buffer引入了复杂性，我们先看第一个例子：本地数据不一致的问题。我们先看看下面的代码：

```java
a = 1;
b = a + 1;
assert b == 2;
```

a和b都是初始化为0，并且变量a在CPU 1的cache line中，变量b在CPU 0的cache line中。

如果CPU执行上述代码，那么第三行的assert不应该失败，不过，如果CPU设计者使用上图中的那个非常简单的store buffer结果，那么你应该会遇到“惊喜”（assert失败了）。具体执行的过程是这样的：

1. CPU 0执行a = 1的赋值操作。
2. CPU 0遇到cache miss。
3. CPU 0发送Read Invalidate消息以便从CPU 1那里获得数据，并Invalid其他CPU保存的本地cache line。
4. CPU 0把要写入的数据“1”放入store buffer。
5. CPU 1收到Read Invalidate后回应，把本地cache line的数据发送给CPU 0并清空本地缓存中a的数据。
6. CPU 0执行b = a + 1。
7. CPU 0收到来自CPU 1的数据，该数据是“0”。
8. CPU 0从cache line中加载a，获得0值。
9. CPU 0将store buffer中的值写入cache line，这时候缓存中a的值是“1”。
10. CPU 0已经在第6步执行了b = a + 1，而当时a还是0，即b的值为实际为1，并已经存储到了cache line中。
11. `assert b == 2;`执行失败。

导致这个问题的根本原因是我们有两个a值，一个在cache line中，一个在store buffer中。

上面这个出错的例子之所以发生是因为它违背了一个基本的原则，即每个CPU按照其视角来观察自己的行为的时候必须是符合程序顺序的。一旦违背这个原则，会导致一些非常不直观的软件行为，对软件工程师而言就是灾难。还好，有“好心”的硬件工程师帮助我们，修改了CPU的设计如下：

![](http://www.wowotech.net/content/uploadfile/201512/d50fa77d45ffd0d7e634799c7c74269a20151210111059.gif)

这种设计叫做store forwarding，**当CPU执行load操作的时候，不但要看缓存，还要看store buffer中是否有内容，如果store buffer有该数据，那么久采用store buffer中的值**。因此即便Store操作还没有写入cahce line，store forwarding的效果看起来就好像CPU的Store操作被向前传递了一样（后面的load的指令可以感知到这个Store操作）。

有了store forwarding的设计，上面的步骤（8）中就可以在store buffer获取正确的a值是“1”而不是“0”，因此计算得到的b的结果就是2，和我们预期的一致了。

### 3、Store buffers and Memory Barriers

关于store buffer引入的复杂性，我们再来看看第二个例子：

```java
public void foo(){
    a = 1;
    b = 1;
}
public void bar(){
    while(b == 0) {
        continue;
    }
    assert a == 1;
}
```

同样的，a和b都是初始化成0。

我们假设CPU 0执行foo函数，CPU 1执行bar函数。我们在进一步假设a变量在CPU 1的缓存中，b在CPU 0的缓存中，执行的额操作序列如下：

1. CPU 0执行a=1的赋值操作，由于a不在本地缓存中，因此，CPU 0将a值放到store buffer中之后，发送了Read Invalidate命令到总线上去。
2. CPU 1执行` while(b == 0)`循环，由于b不再CPU 1的缓存中，因此，CPU发送一个Read消息到总线上，看看是否可以从其他CPU的本地缓存中或者memory中获取数据。
3. CPU 0继续执行b==1的赋值语句，由于b就在自己的本地缓存中（cache line处于modified状态或者exclusive状态），因此CPU 0可以直接操作将新的值1写入cache line。
4. CPU 0收到了Read消息，将最新的b值“1”回送给CPU 1，同时将b所在缓存行的状态迁移为Shard。
5. CPU 1收到了来自CPU 0的Read Response消息，将变量b的最新值“1”写入自己的cache line，并修改状态为Shard。
6. 由于b值等于1了，因此CPU 1跳出while循环，继续前行。
7. CPU 1执行`assert a == 1;`，这时候CPU 1的本地缓存中还是旧的a值，因此断言失败。
8. CPU 1收到了来自CPU 0的Read Invalidate消息，以a变量的值进行回应，同时清空自己的cache line，但是这时已经太晚了。
9. CPU 0收到了Read Response和Invalidate Acknowledge的消息之后，将store buffer中a的最新值“1”数据写入cache line，然并卵，CPU 1已经断言失败了。

遇到这样的问题，CPU设计者也不能直接帮什么忙，毕竟CPU并不知道那些变量有相关性，这些变量是如何相关的。不过CPU设计者可以间接提供一些工具让软件工程师来控制这些相关性。这些工具就是memory barrier指令，要想程序正常运行，必须增加一些memory barrier的操作，具体如下：

```java
public void foo(){
    a = 1;
    smp_mb();
    b = 1;
}
public void bar(){
    while(b == 0) {
        continue;
    }
    assert a == 1;
}
```

`smp_mb()`这个内存屏障的操作会在执行后续的store操作之前，首先flush store buffer（也就是将之前的值写入到cache line中）。`smp_mb()`操作主要是为了让数据在本地缓存中的操作顺序是符合程序顺序的，为了达到这个目标有两种方法：方法一就是让CPU stall，直到完成了清空了store buffer（也就是把store buffer中的数据写入cache line了）。方法二是让CPU可以继续运行，不过需要在store buffer中做些文章，也就是要记录store buffer中数据的顺序，在将store buffer中的数据更新到cache line的操作中，即便是后来的store buffer数据对应的cache line已经读取，也不能执行操作，要等前面的store buffer值写到cache line之后才操作。

增加`smp_mb()`之后的操作顺序如下：

1. CPU 0执行a=1的赋值操作，由于a不在本地缓存中，因此，CPU 0将a值放到store buffer中之后，发送了Read Invalidate命令到总线上去。
2. CPU 1将执行`while(b == 0)`循环，但是，由于b不在CPU 1的缓存中，因此CPU发送一个Read消息到总线上，看看是否可以从其他CPU的本地缓存中或者memory中获取数据。
3. CPU 0执行`smp_mb()`函数，给目前store buffer中的**所有项**做一个标记（后面我们称之为`marked entries`）。当然，针对我们这个例子，store buffer中只有一个marked entry，那就是“a=1”。
4. CPU 0继续执行b=1的赋值语句，虽然b就在自己的本地缓存中（cache line处于Modified状态或者Exclusive状态），不过在store buffer中有marked entry，因此CPU 0并没有直接操作将新的值写入cache line，取而代之的是b的新值“1”被写入store buffer，当然是unmarked状态。
5. CPU 0收到了Read消息，将b值“0”（新值“1”还是store buffer中）回送给CPU 1，同时将b的cache line的状态设定为Shard。
6. CPU 1收到了来自CPU 0的Read Response消息，将b变量的值（“0”）写入自己的cache line，状态修改为Shard。
7. 完成了bus事务之后，CPU 1可以load b到寄存器中了（本地cache line中已经有b值了），当然，这时候b仍然等于0，因此循环不断的loop。虽然b值在CPU 0上已经赋值等于1，但是那个新值被安全的隐藏在CPU 0的storebuffer中。
8. CPU 1收到了来自CPU 0的Read Invalidate消息，以a变量的值进行回应，同时清空自己的cache line。
9. CPU 0将store buffer中的a值写入cache line，并且将caceh line状态修改为Modified状态。
10. 由于store buffer只有一项marked entry（对应a=1）,因此完成step 9之后，store buffer的b也可以进入cache line了。不过需要注意的是，当前b对应的cache line的状态是Shard。
11. CPU 0发送Invalidate消息，请求b数据的独占权。
12. CPU 1收到Invalidate消息，清空自己的b cache line，并回送Acknowledge给CPU 0。
13. CPU 1继续执行`while(b == 0)`循环，由于b不再自己的本地缓存中，因此CPU 1发送Read消息，请求获取b的数据。
14. CPU 0收到Acknowledge消息，将b对应的cache line修改成Exclusive状态，这时候，CPU 0终于可以将b的新值写入cache line。
15. CPU 0收到Read消息，将b的新值1回送给CPU 1，同时将其本地缓存中b对应的cache line状态修改为Shard。
16. CPU 1获取来自CPU 0的b的新值，将其放入cache line中。
17. 由于b值等于1了，因此CPU 1跳出while循环，继续前行。
18. CPU 1执行`assert a == 1`，不过这时候a值没有在自己的cache line中，因此需要通过缓存一致性协议从CPU 0那里获得，这时候获取的a是最新值，也就是1，因此断言成功。

通过上面的描述，我们可以看到，一个直观上很简单的给a变量赋值的操作，都需要那么长的执行过程，而且每一步都需要芯片参与，最终完成整个复杂的操作过程。

## 五、Store Sequence Result in Unnecessary Stalls

不幸的是：每个CPU的store buffer不能实现的太大，其entry的数据不会太多。当CPU以中等的频率执行store操作的时候（假设所有的store操作导致了cache miss），store buffer会很快的被填满。在这种情况下，CPU只能又进入等待状态，直到caceh line完成Invalidate和Acknowledge的交互之后，可以将store buffer的entry写入cache line，从而为新的store让出空间之后，CPU才可以继续执行。这种情况可能发生在调用了memory barrier指令之后，因此一旦store buffer中的某个entry被标记了，那么随后的store都必须等待这个Invalidate完成，因此不管是否cache miss，这些store都必须进入store buffer。

引入invalidate queues可以缓解这个状况。store buffer之所以很容易被填充满，只要是其他CPU回应Invalidate Acknowledge比较慢，如果能够加快这个过程，让store buffer尽快进入cache line，那么也就不会那么容易被填满了。

### 1、Invalidate Queues

Invalidate Acknowledge不能尽快回复的主要原因是Invalidate cache line的操作没有那么快完成，特别是cache比较繁忙的时候，这时，CPU往往进行密集的loading和Storing的操作，而来自其他CPU的，对本CPU本地cache line的操作需要和本CPU的密集的缓存操作进行竞争，只要完了Invalidate操作之后，本CPU才会发生Invalidate Acknowledge。此外，如果短时间内收到大量的Invalidate消息，CPU有可能跟不上处理，从而导致其他CPU不断的等待。

然而，CPU其实不需要完成Invalidate操作就可以回送acknowledge消息，这样，就不会阻止发送Invalidate请求的那个CPU进入无聊的等待状态。CPU可以buffer这些Invalidate消息（放入Invalidate Queues），然后直接回应Acknowledge，表示自己已经收到请求，随后会慢慢处理。当然，再慢也要有一个度，例如对a变量cache line的Invalidate处理必须在该CPU发送任何关于a变量对应的cache line的操作到bus之前完成。

Invalidate Queue的系统结构图如下图所示：

![](http://www.wowotech.net/content/uploadfile/201512/05c956319c3290854e449a73e711a4b220151211113132.gif)

### 2、Invalidate Queues and Invalidate Acknowledge

有了Invalidate Queue的CPU，在收到Invalidate消息的时候首先把它放入Invalidate Queue，同时立刻回送Acknowledge消息，无需等到该cache line被真正的Invalidate之后再回应。当然，如果本CPU想要针对某个cache line向总线发送Invalidate消息的时候，那么CPU必须首先去Invalidate Queue中看看是否有相关的cache line，如果有，那么不能立刻发送，需要等到Invalidate Queue中的cache line被处理完之后再发送。

一旦将一个Invalidate（例如针对变量a的cache line）消息放入CPU的Invalidate Queue，实际上该CPU就等于作出这样的承诺：在处理完该Invalidate消息之前，不会发送任何相关（即针对变量a的cahce line）的MESI协议消息。只要是对该cache line的竞争不是那么激烈，CPU还是对这样的承诺很有信心的。

然而，缓存了Invalidate消息也会引入其他的memory顺序问题，将在下一节讨论。

### 3、Invalidate Queues and Memory Barriers

我们假设CPU缓存Invalidate消息，在操作cache line之前直接回应Invalidate消息。这样的机制对于发送Invalidate的CPU侧是非常好的事，该CPU的store性能会非常高，但是会是内存屏障指令失效，我们来看看下面的例子：

```java
public void foo(){
    a = 1;
    smp_mb();
    b = 1;
}
public void bar(){
    while(b == 0) {
        continue;
    }
    assert a == 1;
}
```

在上面的代码片段中，我们假设a和b初始值都是0，并且a在CPU 0和CPU 1都有缓存副本，即a变量对应的CPU 0和CPU 1的cache line状态都是Shard状态。b处于Exclusive或者Modified状态，被CPU 0独占。我们假设CPU 0执行foo函数，CPU 1执行bar函数。

具体操作如下：





## 六、Read and Weite Memory Barriers

















































