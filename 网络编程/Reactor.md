## [Reactor pattern](https://en.wikipedia.org/wiki/Reactor_pattern)

Reactor设计模式是一种事件处理模式，用于处理由一个或多个输入并发交付给服务处理程序的服务请求。然后，服务处理程序将传入的请求复用，并将它们同步地分派给相关的请求处理程序。

###  [1、结构](https://en.wikipedia.org/wiki/Reactor_pattern#Structure)

- **资源（Resources）**

  可以向系统提供输入或使用系统输出的任何资源。

- **同步事件复用器（Synchronous Event Demultiplexer）**

  使用[event loop](https://en.wikipedia.org/wiki/Event_loop)阻塞所有资源。当可以在资源上启动同步操作而不阻塞时，复用器将资源发送给调度器(*示例：*如果没有数据可读，对[`read()`](https://en.wikipedia.org/wiki/Read_(system_call))的同步调用将阻塞。复用器在资源上使用[`select()`](https://en.wikipedia.org/wiki/Select_(Unix))，该方法会阻塞，直到资源可供读取为止。在这种情况下，对`read()`的同步调用不会阻塞，复用器可以将资源发送给分派器。)

- **分派器（Dispatcher）**

  处理请求处理程序的注册和注销。将资源从复用器分发到关联的请求处理程序。

- **请求处理器（Request Handler）**

  应用程序定义的请求处理程序及其关联资源。

### 2、属性

根据定义，所有Reactor系统都是单线程的，但也可以存在于多线程环境中。

####  [2.1、好处](https://en.wikipedia.org/wiki/Reactor_pattern#Benefits)

Reactor模式将应用程序代码与Reactor的实现完全分离，这意味着应用程序组件可以被分为模块化的、可重用的部分。

#### [2.2、局限](https://en.wikipedia.org/wiki/Reactor_pattern#Limitations)

Reactor模式可能比过程模式更难调试，因为控制流是反向的。此外，通过只同步调用请求处理程序，反应器模式限制了最大并发性，特别是在对称多处理硬件上。反应器模式的可伸缩性不仅受到同步调用请求处理程序的限制，而且还受到复用器的限制。

### [3、另请参阅](https://en.wikipedia.org/wiki/Reactor_pattern#See_also)

- [Proactor pattern](https://en.wikipedia.org/wiki/Proactor_pattern) (一种模式，也可以对事件进行多路复用和分派，但是是异步的)
- [Application server](https://en.wikipedia.org/wiki/Application_server)
- [C10k problem](https://en.wikipedia.org/wiki/C10k_problem)

### [4、参考文件](https://en.wikipedia.org/wiki/Reactor_pattern#References)

1.  Schmidt, Douglas等人. *面向模式的软件体系结构第2卷: 并发和网络化对象的模式.卷2.* Wiley, 2000.
2. Schmidt, Douglas C., [*reactor-siemens*](http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf) (PDF)
3. Kegel, Dan, [*The C10K problem*](http://www.kegel.com/c10k.html#nb.select), retrieved 2007-07-28

### [5、外部链接](https://en.wikipedia.org/wiki/Reactor_pattern#External_links)

- [reactor-siemens](http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf)(PDF) by [Douglas C. Schmidt](https://en.wikipedia.org/wiki/Douglas_C._Schmidt)
- [APR Networking & the Reactor Pattern](http://www.ddj.com/cpp/193101548)
- [高可伸缩的NIO服务器架构 ](https://web.archive.org/web/20100726184112/http://today.java.net/article/2007/02/08/architecture-highly-scalable-nio-based-server)
- [Akka的I/O层架构](http://doc.akka.io/docs/akka/2.2.1/dev/io-layer.html)







# Reactor  

<p align="center">用于同步事件的多路分离和调度句柄的对象行为模式</p>

<p align="center">Douglas C. Schmidt
<p align="center">schmidt@cs.wustl.edu
<p align="center">华盛顿大学计算机科学系，圣路易斯，MO^1

本文的早期版本出现在《程序设计的模式语言》ISBN 0-201-6073-4一书中，该书由Jim Coplien和Douglas C. Schmidt编辑，Addison-Wesley出版，1995年。

## 1、意图

Reactor设计模式处理由一个或多个客户机并发交付给应用程序的服务请求。应用程序中的每个服务可能由几个方法组成，并由一个单独的事件处理程序表示，该事件处理程序负责调度特定于服务的请求。事件处理程序的调度由一个启动调度程序执行，该调度程序管理注册的事件处理程序。服务请求的多路分离由同步事件复用器执行。

## 2、预设名称

调度(Dispatcher)，通知(Notifier)，句柄(Handle)

启动调度器(Initiation Dispatcher)

同步事件复用器(Synchronous Event Demultiplexer)

事件处理程序(Event Handler)
具体事件处理程序(Concrete Event Handlers)

日志接收程序(Logging Acceptor)
日志处理程序(Logging Handler)

## 3、案例

为了说明Reactor模式，请考虑图1中所示的分布式日志记录服务的事件驱动服务器。客户机应用程序使用日志记录服务来记录它们在分布式环境中的状态信息。这些状态信息通常包括错误通知、调试跟踪和性能报告。日志记录被发送到中央日志服务器，该服务器可以将记录写入各种输出设备，如控制台、打印机、文件或网络管理数据库。

![image-20220712150619462](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220712150619462.png)

图1所示的日志服务器处理日志记录和客户机发送的连接请求。日志记录和连接请求可以在多个句柄上并发到达。句柄标识在操作系统中管理的网络通信资源。

日志服务器使用面向连接的协议(如TCP[1])与客户机通信。想要记录数据的客户机必须首先向服务器发送连接请求。服务器使用一个句柄工厂等待这些连接请求，该句柄工厂监听客户机已知的地址。当连接请求到达时，句柄工厂通过创建一个表示连接端点的新句柄在客户机和服务器之间建立连接。这个句柄返回给服务器，然后服务器等待客户机服务请求到达句柄。连接客户机之后，它们可以并发地向服务器发送日志记录。服务器通过连接的套接字句柄接收这些记录。

开发并发日志服务器最直观的方法可能是使用多个线程，这些线程可以并发处理多个客户机，如图2所示。这种方法同步地接受网络连接，并生成“每个连接对应一个线程（thread-per-connection）”来处理客户机日志记录。

![image-20220712151241736](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220712151241736.png)

但是，在服务器端使用多线程实现日志记录处理时，无法解决以下问题:

- **效率**：由于上下文切换、同步和数据移动，线程化可能会导致较差的性能。
- **编程简单**：线程可能需要复杂的并发控制方案。
- **可移植性**：线程并不是在所有操作系统平台上都可用。

由于这些缺点，多线程通常不是开发并发日志服务器的最有效或最简单的解决方案。

## 4、背景

分布式系统中的一种服务器应用程序，它可以并发地从一个或多个客户机接收事件。

## 5、问题

分布式系统中的服务器应用程序必须处理向它们发送服务请求的多个客户机。然而，在调用特定服务之前，服务器应用程序必须将每个传入请求多路分离并调度给相应的服务提供者。开发一种有效的服务器多路分离和调度客户端请求的机制需要解决以下问题:

- **可用性**：服务器必须能够处理传入的请求，即使它正在等待其他请求的到达。特别是，服务器不能在排除其他事件源的情况下无限期地阻塞任何单一事件源的处理，因为这可能会显著地延迟对其他客户端的响应。
- **效率**：服务器必须尽量减少延迟，最大化吞吐量，并避免不必要地使用CPU。
- **编程简单**：服务器的设计应该简化适当并发策略的使用。 
- **适应性**：集成新的或改进的服务(例如更改消息格式或添加服务器端缓存)应该使现有代码的修改和维护成本降到最低。例如，实现新的应用程序服务不应该要求修改通用的事件多路分离和调度机制。
- **可移植性**：将服务器移植到新的操作系统平台不需要太多的努力。

## 6、解决方案

集成事件的同步多路分离和处理事件的相应事件处理程序的调度。此外，将特定于应用程序的服务调度和实现与通用的事件多路分离和调度机制分离开来。

对于应用程序提供的每个服务，引入一个单独的*事件处理程序(Event Handler)*来处理某些类型的事件。所有*事件处理程序(Event Handler)*实现相同的接口。*事件处理程序(Event Handler)*向*启动调度器(Initiation Dispatcher)*注册，该调度器使用*同步事件复用器(Synchronous Event Demultiplexer)*等待事件发生。当事件发生时，*同步事件复用器(Synchronous Event Demultiplexer)*通知*启动调度器(Initiation Dispatcher)*，启动调度器以**同步方式**回调与事件关联的*事件处理程序(Event Handler)*。*事件处理程序(Event Handler)*然后将事件调度给实现所请求服务的方法。

## 7、结构

Reactor模式的主要参与者包括以下成员:

**句柄(Handles)**

识别由操作系统管理的资源。这些资源通常包括网络连接、打开的文件、计时器、同步对象等。在日志服务器中使用句柄(Handles)来标识套接字端点，以便同步事件复用器(Synchronous Event Demultiplexer)可以等待事件在这些端点上发生。日志服务器感兴趣的两种事件类型是连接事件和读取事件，它们分别表示进入的客户端连接和日志数据。日志服务器为每个客户机维护一个单独的连接。在服务器中，每个连接都由套接字句柄(handle)表示。

**同步事件复用器(Synchronous Event Demultiplexer)**  

阻塞等待事件发生在一组句柄(Handles)上。当可以在不阻塞的情况下初始化句柄(Handle)上的操作时，它将返回。常用的I/O事件多路分离是`select`[1]，它是UNIX和Win32操作系统平台提供的事件多路分离系统调用。`select`调用指示哪些句柄(Handles)可以在不阻塞应用程序进程的情况下同步调用操作。

**启动调度器(Initiation Dispatcher)**

定义用于注册、删除和调度事件处理程序(Event Handlers)的接口。最终，同步事件复用器(Synchronous Event Demultiplexer)负责等待，直到新的事件发生。当它检测到新的事件时，它通知启动调度器(Initiation Dispatcher)回调应用程序特定的事件处理程序。常见事件包括连接接受事件、数据输入输出事件和超时事件。

**事件处理器(Event Handler)**

指定一个由钩子方法[3]组成的接口，该方法抽象地表示特定于服务的事件的调度操作。此方法必须由特定于应用程序的服务实现。

**具体事件处理器(Concrete Event Handler)**

实现钩子方法，以及以特定于应用程序的方式处理这些事件的方法。应用程序向启动调度器(Initiation Dispatcher)注册具体事件处理程序(Concrete Event Handler)以处理某些类型的事件。当这些事件到达时，启动调度器(Initiation Dispatcher)回调相应的具体事件处理程序(Concrete Event Handler)的钩子方法。 

日志服务器中有两个具体事件处理程序(Concrete Event Handlers)：日志处理程序(Logging Handler)和日志接收程序(Logging Acceptor)。日志处理程序(Logging Handler)负责接收和处理日志记录。日志接收程序(Logging Acceptor)创建并连接日志处理程序，这些处理程序处理来自客户机的后续日志记录。

Reactor模式参与者的结构如下OMT类图所示：

![image-20220711111914658](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220711111914658.png)

## 8、动态

### 8.1、一般协作  

以下协作发生在Reactor模式中：

- 当应用程序向启动调度器(Initiation Dispatcher)注册具体事件处理程序(Concrete Event Handler)时，应用程序指示该事件处理程序(Event Handler)希望启动调度器(Initiation Dispatcher)通知它的事件类型——该事件何时在关联的句柄(Handle)上发生。
- 启动调度器(Initiation Dispatcher)请求每个事件处理程序(Event Handler)返回其内部句柄(Handle)。此句柄(Handle)标识给操作系统的事件处理程序(Event Handler)。
- 注册所有事件处理程序(Event Handler)之后，应用程序调用处理事件来启动启动调度器(Initiation Dispatcher)的事件循环。此时，启动调度器(Initiation Dispatcher)将来自每个注册事件处理程序(Event Handler)的句柄(Handle)组合在一起，并使用同步事件复用器(Synchronous Event Demultiplexer)等待事件在这些句柄(Handle)上发生。例如，TCP协议层使用select同步事件多路分离操作来等待客户端日志记录事件到达连接的套接字句柄(Handles)。
- 当与事件源对应的句柄“就绪(ready)”时，同步事件复用器(Synchronous Event Demultiplexer)通知启动调度器(Initiation Dispatcher)，例如，TCP套接字“准备好读取了(ready for reading)”。
- 启动调度器(Initiation Dispatcher)触发事件处理程序(Event Handler)钩子方法以响应就绪句柄(Handle)上的事件。当事件发生时，启动调度器(Initiation Dispatcher)使用事件源激活的句柄(Handle)作为“键(keys)”来定位和调度适当的事件处理程序(Event Handler)的钩子方法。
- 启动调度器(Initiation Dispatcher)回调事件处理程序(Event Handler)的handle_event钩子方法，以执行特定于应用程序的功能来响应事件。所发生的事件类型可以作为参数传递给该方法，并由该方法在内部使用，以执行额外的特定于服务的多路分离和调度。另一种调度方法将在第9.4节中介绍。

下面的交互图说明了在Reactor模式下，应用程序代码和参与者之间的协作：

![image-20220711112327341](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220711112327341.png)

### 8.2、协作场景

日志服务器的Reactor模式中的协作可以用两种场景来说明。这些场景展示了使用响应式事件调度设计的日志服务器如何处理来自多个客户机的连接请求和日志数据。

#### 8.2.1、客户机连接响应式日志服务器  

第一个场景展示了客户机连接到日志服务器时所采取的步骤。

![image-20220711112416731](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220711112416731.png)

这一系列的步骤可以总结如下：

1. 日志服务器(1)向启动调度器(Initiation Dispatcher)注册日志接收器(Logging Acceptor)来处理连接请求;
2. 日志服务器调用启动调度器(Initiation Dispatcher)的handle_events()方法(2);
3. 启动调度器(Initiation Dispatcher)调用同步事件多路分离select(3)操作来等待连接请求或日志数据的到达;
4. 客户机连接(4)到日志服务器;
5. 日志接收器(Logging Acceptor)被启动调度器(Initiation Dispatcher)(5)通知新的连接请求;
6. 日志接收器(Logging Acceptor)接收(6)新的连接;
7. 日志接收器(Logging Acceptor)创建(7)一个日志处理器(Logging Handler)来服务新客户机;
9. 日志处理程序(Logging Handler)向启动调度器(Initiation Dispatcher)注册它的套接字句柄，并指示调度程序在套接字“准备读取(ready for reading)”时通知它。

#### 8.2.2、客户机发送日志记录到响应日志服务器  

第二个场景展示了响应式日志记录服务器为服务日志记录所采取的步骤序列。

![image-20220711112654729](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220711112654729.png)

这一系列的步骤可以总结如下:

1. 客户机发送(1)一个日志记录;
2. 当一个客户机日志记录被操作系统排队到它的套接字句柄上时，启动调度器(Initiation Dispatcher)通知(2)相关的日志处理程序(Logging Handler);
3. 以非阻塞的方式接收(3)记录（步骤2和3重复，直到完全接收到日志记录）;
4. 日志处理程序(Logging Handler)处理日志记录并将(4)它写入标准输出。
5. 日志处理程序(Logging Handler)将(5)控制权返回给启动调度器(Initiation Dispatcher)的事件循环。

## 9、实现

本节描述如何在c++中实现Reactor模式。下面描述的实现受到ACE通信软件框架[2]中提供的可重用组件的影响。

### 9.1、选择同步事件复用器机制

启动调度器(Initiation Dispatcher)使用同步事件复用器(Synchronous Event Demultiplexer)以同步方式等待，直到一个或多个事件发生。这通常是通过一个操作系统事件多路分离系统调用(如`select`)来实现的。`select`调用表示哪些句柄(Handle)准备好执行I/O操作，而不会阻塞特定于应用程序的服务处理程序所在的操作系统进程。通常，同步事件复用器(Synchronous Event Demultiplexer)基于现有的操作系统机制，而不是由Reactor模式的实现者开发的。

### 9.2、开发一个启动调度器

以下是开发启动调度器(Initiation Dispatcher)所需的步骤：

**实现事件处理程序(Event Handler)表**：启动调度器(Initiation Dispatcher)维护一个具体事件处理程序表(Concrete Event Handlers)。因此，启动调度器(Initiation Dispatcher)提供了在运行时从该表注册和删除处理程序的方法。这个表可以通过各种方式实现，例如，使用哈希、线性搜索，或者如果句柄表示为连续的小整数值范围，则可以直接索引。

**实现事件循环入口点**：启动调度器(Initiation Dispatcher)的事件循环的入口点应该由一个`handle_events`方法提供。此方法控制同步事件复用器(Synchronous Event Demultiplexer)提供的句柄(Handle)多路分离，以及执行事件处理程序(Event Handler)调度。通常，整个应用程序的主事件循环由这个入口点控制。

当事件发生时，启动调度器(Initiation Dispatcher)从同步事件复用器(Synchronous Event Demultiplexer)的调用返回，并通过为每个“准备好(ready)”的句柄(handle)调度事件处理程序(Event Handler)的句柄`event_hook`方法来“作出反应(reacts)”。此钩子方法执行用户定义的代码，并在完成时将控制权返回给启动调度器(Initiation Dispatcher)。

下面的c++类说明了启动调度器(Initiation Dispatcher)公共接口的核心方法:

```c++
enum Event_Type
    // = TITLE
    // Types of events handled by the
    // Initiation_Dispatcher.
    //
    // = DESCRIPTION
    // These values are powers of two so
    // their bits can be efficiently ‘‘or’d’’
    // together to form composite values.
{
    ACCEPT_EVENT = 01,
    READ_EVENT = 02,
    WRITE_EVENT = 04,
    TIMEOUT_EVENT = 010,
    SIGNAL_EVENT = 020,
    CLOSE_EVENT = 040
};
class Initiation_Dispatcher
    // = TITLE
    // Demultiplex and dispatch Event_Handlers
    // in response to client requests.
{
    public:
    // Register an Event_Handler of a particular
    // Event_Type (e.g., READ_EVENT, ACCEPT_EVENT,
    // etc.).
    int register_handler (Event_Handler *eh,
                          Event_Type et);
    // Remove an Event_Handler of a particular
    // Event_Type.
    int remove_handler (Event_Handler *eh,
                        Event_Type et);
    // Entry point into the reactive event loop.
    int handle_events (Time_Value *timeout = 0);
};
```

**实现必要的同步机制**：如果在只有一个线程控制的应用程序中使用Reactor模式，就有可能消除所有同步。在这种情况下，启动调度器(Initiation Dispatcher)串行化应用程序进程中的事件处理程序(Event Handler)句柄event_hooks。

然而，在多线程应用程序中，启动调度器(Initiation Dispatcher)也可以充当中心事件调度器。在这种情况下，修改或激活共享状态变量(例如保存事件处理程序(Event Handler)的表)时，必须串行化启动调度器(Initiation Dispatcher)中的关键部分，以防止竞态条件。防止竞争条件的常用技术是使用互斥机制，如信号量或互斥变量。

为了防止自死锁，互斥机制可以使用递归锁[4]。递归锁持有可以防止由同一线程在启动调度器(Initiation Dispatcher)中跨事件处理程序(Event Handler)钩子方法持有锁时发生死锁。拥有递归锁的线程可以重新获得该锁，而不会阻塞该线程。这个属性很重要，因为Reactor的`handle_events`方法会回调应用程序特定的事件处理程序(Event Handler)。应用程序钩子方法代码随后可能会通过它的注册处理程序重新进入启动调度器(Initiation Dispatcher)，并删除处理程序方法。

### 9.3、确定调度目标的类型

可以将两种不同类型的事件处理程序(Event Handler)与一个句柄(Handle)关联起来，作为启动调度器(Initiation Dispatcher)的调度逻辑的目标。Reactor模式的实现可以选择以下调度方案中的一种或两种：

**事件处理程序(Event Handler)对象**：将事件处理程序(Event Handler)与句柄(Handle)关联的一种常见方法是使事件处理程序(Event Handler)成为一个对象。例如，第7节中展示的Reactor模式实现将事件处理程序(Event Handler)的子类对象注册为启动调度器(Initiation Dispatcher)。使用对象作为调度目标可以方便地子类化事件处理程序(Event Handler)，以便重用和扩展现有组件。此外，对象将服务的状态和方法集成到单个组件中。

**事件处理程序(Event Handler)函数**：将事件处理程序(Event Handler)与句柄(Handle)关联的另一种方法是向启动调度器(Initiation Dispatcher)注册函数。使用函数作为调度目标使注册回调变得很方便，而不必定义从事件处理程序(Event Handler)继承的新类。

可以使用适配器模式[5]同时支持对象和函数。例如，可以使用一个事件处理程序对象来定义适配器，该对象保存一个指向事件处理程序函数的指针。在事件处理程序适配器对象上调用`handle_event`方法时，它可以自动将调用转发给它持有的事件处理程序函数。

### 9.4、定义事件处理接口

假设我们使用事件处理程序(Event Handler)对象而不是函数，下一步是定义事件处理程序(Event Handler)的接口。有两种方法:

**单一方法的接口**：第7节中的OMT图演示了事件处理程序(Event Handler)基类接口的实现，该接口包含一个名为`handle_event`的方法，启动调度器(Initiation Dispatcher)使用该方法来调度事件。在这种情况下，已发生事件的类型作为参数传递给方法。

下面的c++抽象基类说明了单一方法接口：

```c++
class Event_Handler
    // = TITLE
    // Abstract base class that serves as the
    // target of the Initiation_Dispatcher.
{
    public:
    // Hook method that is called back by the
    // Initiation_Dispatcher to handle events.
    virtual int handle_event (Event_Type et) = 0;
    // Hook method that returns the underlying
    // I/O Handle.
    virtual Handle get_handle (void) const = 0;
}; 
```

单一方法接口的优点是可以在不改变接口的情况下添加新的事件类型。然而，这种方法鼓励在子类的`handle_event`方法中使用`switch`语句，这限制了它的可扩展性。

**多方法接口**：实现事件处理程序(Event Handler)接口的另一种方法是为每种类型的事件(如`handle_input`、`handle_output`或`handle_timeout`)定义单独的虚拟钩子方法。

下面的c++抽象基类说明了多方法接口：

```c++
class Event_Handler
{
    public:
    // Hook methods that are called back by
    // the Initiation_Dispatcher to handle
    // particular types of events.
    virtual int handle_accept (void) = 0;
    virtual int handle_input (void) = 0;
    virtual int handle_output (void) = 0;
    virtual int handle_timeout (void) = 0;
    virtual int handle_close (void) = 0;
    // Hook method that returns the underlying
    // I/O Handle.
    virtual Handle get_handle (void) const = 0;
};
```

多方法接口的好处是，它很容易有选择地重写基类中的方法，并避免进一步的多路分离，例如，在钩子方法中通过`switch`或`if`语句。但是，它要求框架开发人员提前预测事件处理程序(Event Handler)方法集。例如，上面事件处理程序(Event Handler)接口中的各种`handle_*`方法是针对UNIX选择(`select`)机制中可用的I/O事件定制的。然而，这个接口还不够广泛，不足以包含通过Win32 `WaitForMultipleObjects`机制[6]处理的所有事件类型。

上面描述的两种方法都是[3]中描述的钩子方法模式和[7]中描述的工厂回调模式的例子。这些模式的目的是提供定义良好的钩子，这些钩子可以由应用程序专门化，并由低级调度代码回调。

### 9.5、确定应用程序中启动调度程序的数量

许多应用程序只需使用Reactor模式的一个实例就可以结构化。在这种情况下，启动调度器(Initiation Dispatcher)可以实现为单例(Singleton)[5]。这种设计对于将事件多路分离和调度到应用程序中的单个位置非常有用。

但是，一些操作系统限制了单个控制线程中可等待的句柄(Handle)数量。例如，Win32允许`select`和`WaitForMultipleObjects`在一个线程中等待不超过`64`个句柄(Handle)。在这种情况下，可能需要创建多个线程，每个线程都运行自己的Reactor模式实例。

注意，事件处理程序(Event Handler)只在Reactor模式的实例中串行化。因此，多个线程中的多个事件处理程序(Event Handler)可以并行运行。如果不同线程中的事件处理程序(Event Handler)访问共享状态，这种配置可能需要使用额外的同步机制。

### 9.6、实现具体事件处理程序

具体的事件处理程序(Concrete Event Handlers)通常由应用程序开发人员创建，以执行特定的服务来响应特定的事件。当启动调度程序(Initiation Dispatcher)调用相应的钩子方法时，开发人员必须确定执行什么处理。

下面的代码实现了第3节中描述的日志服务器的具体事件处理程序(Concrete Event Handler)。这些处理程序提供被动连接建立(日志接收程序(Logging Acceptor))和数据接收(日志处理程序(Logging Handler))。

**日志接收程序(Logging Acceptor)类**：该类是Acceptor-Connector模式[8]的Acceptor组件的一个示例。Acceptor-Connector模式将服务初始化任务与服务初始化后执行的任务分离开来。此模式允许服务中特定于应用程序的部分，如日志处理程序(Logging Handler)，独立于用于建立连接的机制而变化。

日志接收程序(Logging Acceptor)被动地接受来自客户机应用程序的连接，并创建特定于客户机的日志处理程序(Logging Handler)对象，该对象接收和处理来自客户机的日志记录。日志接收程序(Logging Acceptor)类中的关键方法和数据成员定义如下:

```c++
class Logging_Acceptor ：public Event_Handler
    // = TITLE
    // Handles client connection requests.
{
    public:
    // Initialize the acceptor_ endpoint and
    // register with the Initiation Dispatcher.
    Logging_Acceptor (const INET_Addr &addr);
    // Factory method that accepts a new
    // SOCK_Stream connection and creates a
    // Logging_Handler object to handle logging
    // records sent using the connection.
    virtual void handle_event (Event_Type et);
    // Get the I/O Handle (called by the
    // Initiation Dispatcher when
    // Logging_Acceptor is registered).
    virtual HANDLE get_handle (void) const
    {
        return acceptor_.get_handle ();
    }
    private:
    // Socket factory that accepts client
    // connections.
    SOCK_Acceptor acceptor_;
};
```

日志接收程序(Logging Acceptor)类继承自事件处理程序(Event Handler)基类。这使应用程序能够向启动调度程序(Initiation Dispatcher)注册日志接收程序(Logging Acceptor)。

日志接收程序(Logging Acceptor)还包含SOCK Acceptor的一个实例。这是一个具体的工厂，它使日志接收程序(Logging Acceptor)能够在监听通信端口的被动模式套接字上接受连接请求。当一个连接从客户端到达时，SOCK Acceptor接受该连接并生成一个SOCK Stream对象。从此以后，SOCK Stream对象被用来在客户端和日志服务器之间可靠地传输数据。

用于实现日志服务器的SOCK Acceptor和SOCK Stream类是ACE[9]提供的c++套接字包装器库的一部分。这些套接字包装器将套接字接口的SOCK Stream语义封装在一个可移植的、类型安全的面向对象接口中。在Internet领域，SOCK Stream套接字是使用TCP实现的。

日志接收器(Logging Acceptor)的构造函数为`ACCEPT`事件向启动调度器单例(Initiation Dispatcher Singleton)[5]注册自己，如下所示：

```c++
Logging_Acceptor::Logging_Acceptor
    (const INET_Addr &addr)
    ：acceptor_ (addr)
    {
        // Register acceptor with the Initiation
        // Dispatcher, which "double dispatches"
        // the Logging_Acceptor::get_handle() method
        // to obtain the HANDLE.
        Initiation_Dispatcher::instance ()->
            register_handler (this, ACCEPT_EVENT);
    }
```

至此以后，每当客户端连接到达时，启动调度器(Initiation Dispatcher)回调日志接收器(Logging Acceptor)的`handle_event`方法，如下所示:

```c++
void
    Logging_Acceptor::handle_event (Event_Type et)
{
    // Can only be called for an ACCEPT event.
    assert (et == ACCEPT_EVENT);
    SOCK_Stream new_connection;
    // Accept the connection.
    acceptor_.accept (new_connection);
    // Create a new Logging Handler.
    Logging_Handler *handler =
        new Logging_Handler (new_connection);
}
```

`handle_event`方法调用SOCK Acceptor的`accept`方法来被动地建立SOCK Stream。SOCK Stream与新客户机连接之后，将动态分配一个日志处理程序(Logging Handler)来处理日志记录请求。如下所示，日志处理程序(Logging Handler)将自己注册到启动调度器(Initiation Dispatcher)，它将其关联客户机的所有日志记录多路分离到它。

**日志处理程序(Logging Handler)类**：日志服务器使用如下所示的日志处理程序(Logging Handler)类来接收客户端应用程序发送的日志记录：

```c++
class Logging_Handler ：public Event_Handler
    // = TITLE
    // Receive and process logging records
    // sent by a client application.
{
    public:
    // Initialize the client stream.
    Logging_Handler (SOCK_Stream &cs);
    // Hook method that handles the reception
    // of logging records from clients.
    virtual void handle_event (Event_Type et);
    // Get the I/O Handle (called by the
    // Initiation Dispatcher when
    // Logging_Handler is registered).
    virtual HANDLE get_handle (void) const
    {
        return peer_stream_.get_handle ();
    }
    private:
    // Receives logging records from a client.
    SOCK_Stream peer_stream_;
}; 
```

日志处理程序(Logging Handler)继承自事件处理程序(Event Handler)，这使它能够注册到启动调度器(Initiation Dispatcher)，如下所示：

```c++
Logging_Handler::Logging_Handler
    (SOCK_Stream &cs)
    ：peer_stream_ (cs)
    {
        // Register with the dispatcher for
        // READ events.
        Initiation_Dispatcher::instance ()->
            register_handler (this, READ_EVENT);
    }
```

一旦创建了Logging Handler，Logging Handler就会向Initiation Dispatcher Singleton注册`READ`事件。此后，当日志记录到达时，Initiation Dispatcher自动调度相关的Logging Handler的`handle_event`方法，如下所示:

```c++
void
    Logging_Handler::handle_event (Event_Type et)
{
    if (et == READ_EVENT) {
        Log_Record log_record;
        peer_stream_.recv ((void *) log_record, sizeof log_record);
        // Write logging record to standard output.
        log_record.write (STDOUT);
    }
    else if (et == CLOSE_EVENT) {
        peer_stream_.close ();
        delete (void *) this;
    }
}
```

当套接字句柄(Handle)上发生`READ`事件时，Initiation Dispatcher回调Logging Handler的`handle_event`方法。这个方法接收、处理日志记录，并将其写入标准输出(`STDOUT`)。同样，当客户端关闭连接时，Initiation Dispatcher传递一个`CLOSE`事件，通知Logging Handler关闭它的SOCK流并删除自己。

### 9.7、实现服务器

日志服务器包含一个main函数。

**日志服务器main函数**：该函数实现了一个单线程的、并发的日志服务器，它在Initiation Dispatcher的`handle_events`事件循环中等待。当请求从客户机到达时，Initiation Dispatcher调用适当的Concrete Event Handler钩子方法，这些方法接受连接并接收和处理日志记录。进入日志服务器的主入口点定义如下：

```c++
// Server port number.
const u_short PORT = 10000;
int
    main (void)
{
    // Logging server port number.
    INET_Addr server_addr (PORT);
    // Initialize logging server endpoint and
    // register with the Initiation_Dispatcher.
    Logging_Acceptor la (server_addr);
    // Main event loop that handles client
    // logging records and connection requests.
    for (;;)
        Initiation_Dispatcher::instance ()->
        handle_events ();
    /* NOTREACHED */
    return 0;
}
```

主程序创建一个Logging Acceptor，它的构造函数用日志服务器的端口号初始化它。然后程序进入它的主事件循环。随后，Initiation Dispatcher Singleton使用`select`事件多路分离系统调用来同步等待来自客户机的连接请求和日志记录。

下面的交互图说明了参与日志服务器示例的对象之间的协作:

![image-20220712180859930](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220712180859930.png)

初始化Initiation Dispatcher对象之后，它就成为日志服务器中控制流的主要焦点。所有后续的活动都是由向Initiation Dispatcher注册并受其控制的Logging Acceptor和Logging Handler对象上的钩子方法触发的。

当连接请求到达网络连接时，Initiation Dispatcher回调Logging Acceptor，后者接受网络连接并创建一个Logging Handler。然后，这个Logging Handler向Initiation Dispatcher注册READ事件。因此，当客户端发送日志记录时，Initiation Dispatcher回调客户端的Logging Handler，以在日志服务器的单控制线程中处理来自该客户端连接的传入记录。

## 10、已知应用

Reactor模式已经在许多面向对象的框架中使用，包括以下框架:

- **InterViews**：Reactor模式由InterViews [10] window系统分布实现，在这里它被称为调度器(Dispatcher)。InterViews调度器(Dispatcher)用于定义应用程序的主事件循环，并管理一个或多个物理GUI显示的连接。
- **ACE Framework**：ACE框架[11]使用Reactor模式作为其中心事件多路分离器和调度器。

Reactor模式已被用于许多商业项目，包括:

- **CORBA ORBs**：CORBA[12]的许多单线程实现(如VisiBroker、Orbix和TAO[13])中的ORB核心层使用Reactor模式多路分离并将ORB请求调度给服务。
- **爱立信EOS呼叫中心管理系统**：该系统使用Reactor模式管理呼叫中心管理系统中的事件服务器[14]在PBX和管理员之间路由的事件。
- **Project Spectrum**：Spectrum[15]项目的高速医学图像传输子系统在医学成像系统中采用了Reactor模式。

## 11、影响

### 11.1、好处

反应器模式提供了以下好处：

- **关注分离**：Reactor模式将独立于应用程序的多路分离和调度机制与特定于应用程序的钩子方法功能分离开来。与应用程序无关的机制成为可重用的组件，它们知道如何多路分离事件和调度由事件处理程序(Event Handlers)定义的适当钩子方法。相反，钩子方法中的特定于应用程序的功能知道如何执行特定类型的服务。
- **改进事件驱动应用程序的模块化、可重用性和可配置性**：该模式将应用程序功能分离到单独的类中。例如，日志服务器中有两个独立的类：一个用于建立连接，另一个用于接收和处理日志记录。这种解耦允许对不同类型的面向连接的服务(如文件传输、远程登录和视频点播)重用连接建立类。因此，修改或扩展日志服务器的功能只会影响日志处理程序类的实现。
- **提高应用程序的可移植性**：可以独立于执行事件多路分离的操作系统调用重用启动调度器(Initiation Dispatcher)接口。这些系统调用检测并报告一个或多个事件的发生，这些事件可能同时发生在多个事件源上。常见的事件来源可能包括I/O句柄、计时器和同步对象。在UNIX平台上，事件多路分离系统调用称为select和poll[1]。在Win32 API[16]中，WaitForMultipleObjects系统调用执行事件多路分离。
- **提供粗粒度并发控制**：Reactor模式在进程或线程内的事件多路分离和调度级别串行化事件处理程序的调用。在启动调度器(Initiation Dispatcher)级别进行串行化通常可以消除应用程序进程中更复杂的同步或锁定的需要。

### 11.2、缺陷

Reactor模式有以下缺陷：

- **适用性受限**：只有当操作系统支持句柄(Handles)时，才能有效地应用Reactor模式。在启动调度器(Initiation Dispatcher)中使用多个线程来模拟Reactor模式的语义是可能的，例如，每个句柄(Handle)一个线程。每当句柄上有可用的事件时，其关联的线程就会读取该事件，并将其放置到启动调度器(Initiation Dispatcher)按顺序处理的队列中。但是，这种设计通常效率很低，因为它串行化所有事件处理程序(Event Handlers)，从而增加了同步和上下文切换开销，而没有增强并行性。
- **非抢占**：在单线程应用程序进程中，事件处理程序(Event Handlers)在执行时不会被抢占。这意味着事件处理程序(Event Handlers)不应该对单个句柄(Handle)执行阻塞I/O，因为这将阻塞整个进程，并阻碍连接到其他句柄(Handles)的客户端的响应。因此，对于长持续时间的操作，例如传输多兆字节的医学图像[15]，活动对象(Active Object)模式[17]可能更有效。活动对象(Active Object)使用多线程或多处理来与启动调度器(Initiation Dispatcher)的主事件循环并行地完成它的任务。 
- **难以调试**：使用Reactor模式编写的应用程序可能很难调试，因为反向控制流在框架基础设施和应用程序特定处理程序的方法回调之间来回振荡。这增加了在调试器中通过框架运行时行为进行“单步执行(single-stepping)”的难度，因为应用程序开发人员可能无法理解或访问框架代码。这类似于调试用LEX和YACC编写的编译器词法分析器和解析器时遇到的问题。在这些应用程序中，当控制线程位于用户定义的操作例程中时，调试很简单。然而，一旦控制线程返回到生成的确定性有限自动机(Deterministic Finite Automata, DFA)框架，就很难遵循程序逻辑。

## 12、另请参阅

Reactor模式与观察者(Observer)模式[5]相关，在观察者(Observer)模式中，当单个主题(subject)发生变化时，所有依赖项都会被通知。在Reactor模式中，当事件源上发生与处理程序相关的事件时，单个处理程序将得到通知。观察者(Observer)模式通常用于将事件从多个源多路分离到其关联的事件处理程序，而观察者通常只与单个事件源关联。

Reactor模式与责任链(CoR)模式[5]相关，在责任链(CoR)模式模式中，请求被委托给负责的服务提供者。Reactor模式与CoR模式不同，因为Reactor将特定的事件处理程序与特定的事件源关联起来，而CoR模式则搜索链以定位第一个匹配的事件处理程序。

可以认为Reactor模式是异步Proactor模式[18]的同步变体。Proactor支持多个事件处理程序的多路分离和调度，这些事件处理程序是由异步事件完成触发的。相反，Reactor模式负责多路分离和调度多个事件处理程序，当可以同步启动一个操作而不阻塞时触发这些处理程序。

活动对象(Active Object)模式[17]将方法执行与方法调用分离开来，以简化在不同控制线程中调用的方法对共享资源的同步访问。当线程不可用时，或者线程的开销和复杂性不受欢迎时，通常使用Reactor模式来代替活动对象模式。

Reactor模式的实现为事件多路分离提供了Facade[5]。Facade是一个接口，它保护应用程序不受子系统中复杂对象关系的影响。
