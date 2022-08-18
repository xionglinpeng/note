# Netty



## Netty架构模型

![](https://pic3.zhimg.com/80/v2-cc6bf4ea10291ba5f51d469b49d25a4e_1440w.jpg)

## 配置ServerBootstrap

## NioEventLoopGroup

![]()

### 类简介

<font>Interface</font>

`EventExecutorGroup`：负责通过next()方法提供EventExecutor。除此之外，它还负责处理它们的生命周期，并允许以全局方式关闭它们，提供了shutdown，termination等方法。由于EventExecutorGroup还继承了java.util.concurrent.ScheduledExecutorService接口，因此EventExecutorGroup还提供了任务执行/调度等行为。

`EventLoopGroup`：特殊的EventExecutorGroup，提供了通道的注册操作，以便在事件循环期间进行后续选择。

<font>Class</font>

`AbstractEventExecutorGroup`：对EventExecutorGroup接口的实现，实现了EventExecutorGroup接口中的submit，schedule，execute，invoke等系列方法。

`MultithreadEventExecutorGroup`：EventExecutorGroup实现的抽象基类，构造方法中完成了对EventLoop线程组的初始化。同时实现了next()方法，以及shutdown和Termination相关方法。next()方法中的EventLoop选择器（DefaultEventExecutorChooserFactory）在MultithreadEventExecutorGroup的构造方法中初始化

`MultithreadEventLoopGroup`：EventLoopGroup实现的抽象基类，实现了EventLoopGroup接口中的register方法，同时提供了EventLoopGroup中EventLoop线程的默认数量。

`NioEventLoopGroup`：MultithreadEventLoopGroup实现，实现MultithreadEventLoopGroup的newChild方法提供对EventLoop实例的创建，同时在构造中提供了默认的队列拒绝策略（RejectedExecutionHandlers）和seletor选择策略（DefaultSelectStrategyFactory）。

### 详细描述

#### EventExecutorGroup



#### EventLoopGroup

```java
public interface EventLoopGroup extends EventExecutorGroup {
    @Override
    EventLoop next();
    ChannelFuture register(Channel channel);
    ChannelFuture register(ChannelPromise promise);
    @Deprecated
    ChannelFuture register(Channel channel, ChannelPromise promise);
}
```

#### AbstractEventExecutorGroup

查看类中的方法实现，可以发现，所有的方法都通过next()方法转调。而next()方法获取的就是EventLoop，类型为EventExecutor。也就是说，AbstractEventExecutorGroup的实现，只是一个委派操作，其具体的操作由EventLoop完成。

#### MultithreadEventExecutorGroup

MultithreadEventExecutorGroup包含五个成员变量，如下代码所示：

```java
public abstract class MultithreadEventExecutorGroup extends AbstractEventExecutorGroup {
    private final EventExecutor[] children;
    private final Set<EventExecutor> readonlyChildren;
    private final AtomicInteger terminatedChildren = new AtomicInteger();
    private final Promise<?> terminationFuture = new DefaultPromise(GlobalEventExecutor.INSTANCE);
    private final EventExecutorChooserFactory.EventExecutorChooser chooser;
    ...
}
```

变量说明：

1. `children`：EventExecutor线程数组，每一个EventExecutor代表一个NioEventLoop。将在MultithreadEventExecutorGroup构造函数中被初始化。
2. `readonlyChildren`：只读EventExecutor集合，与`children`一致。
3. `terminatedChildren`：
4. `terminationFuture`：
5. `chooser`：EventExecutor选择器，Netty默认提供了两种选择算法（具体请参考xxx），选其中在MultithreadEventExecutorGroup构造函数中被设置，类型为DefaultEventExecutorChooserFactory。

<font>EventLoop初始化</font>

```java
protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                        EventExecutorChooserFactory chooserFactory, Object... args) {
    checkPositive(nThreads, "nThreads");

    if (executor == null) {
        executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
    }
	// 创建EventExecutor数组，数量nThreads个
    children = new EventExecutor[nThreads];
	//迭代nThreads次
    for (int i = 0; i < nThreads; i ++) {
        boolean success = false;
        try {
            //通过抽象方法newChild创建EventExecutor实例（具体实现由NioEventLoopGroup完成）
            children[i] = newChild(executor, args);
            success = true;
        } catch (Exception e) {
            // TODO: Think about if this is a good exception type
            throw new IllegalStateException("failed to create a child event loop", e);
        } finally {
            //如果创建EventExecutor实例失败，将进入下面的if代码块，
            //然后将前面已经创建成功的EventExecutor实例全部优雅地关闭
            if (!success) {
                for (int j = 0; j < i; j ++) {
                    children[j].shutdownGracefully();
                }

                for (int j = 0; j < i; j ++) {
                    EventExecutor e = children[j];
                    try {
                        while (!e.isTerminated()) {
                            e.awaitTermination(Integer.MAX_VALUE, TimeUnit.SECONDS);
                        }
                    } catch (InterruptedException interrupted) {
                        // Let the caller handle the interruption.
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            }
        }
    }
	
    //通过EventExecutorChooserFactory创建一个EventExecutor选择器实例
    chooser = chooserFactory.newChooser(children);
	//创建一个终止监听器
    final FutureListener<Object> terminationListener = new FutureListener<Object>() {
        @Override
        public void operationComplete(Future<Object> future) throws Exception {
            if (terminatedChildren.incrementAndGet() == children.length) {
                //所有都被终止完成，将success设为NULL
                terminationFuture.setSuccess(null); 
            }
        }
    };
	//为所有EventExecutor添加终止监听器
    for (EventExecutor e: children) {
        e.terminationFuture().addListener(terminationListener);
    }
	//将EventExecutor数组（children）转换成不可变集合，并赋值给变量readonlyChildren
    Set<EventExecutor> childrenSet = new LinkedHashSet<EventExecutor>(children.length);
    Collections.addAll(childrenSet, children);
    readonlyChildren = Collections.unmodifiableSet(childrenSet);
}
```

EventExecutor数组的初始化位于MultithreadEventExecutorGroup的构造函数中，包含四个参数：

1. `int nThreads`：group的线程数量，也代表着EventExecutor的数量。
2. `Executor executor`：？？？
3. `EventExecutorChooserFactory chooserFactory`：EventExecutor选择器工厂，通过此工厂类创建EventExecutor选择器。
4. `Object... args`：用于创建EventExecutor所需的参数。

<font>shutdown和Termination</font>

MultithreadEventExecutorGroup对EventExecutorGroup接口中shutdown和Termination的实现是实际上是交由EventExecutor处理，不过由于MultithreadEventExecutorGroup是管理多个EventExecutor，因此，在MultithreadEventExecutorGroup中，是通过轮询的方式依次调用EventExecutor进行shutdown或Termination。

其中一个shutdown函数：

```java
@Override
public boolean isShutdown() {
    for (EventExecutor l: children) {
        if (!l.isShutdown()) {
            return false;
        }
    }
    return true;
}
```

#### MultithreadEventLoopGroup

MultithreadEventLoopGroup只完成了两个工作：

1. 实现EventLoopGroup接口中的register方法，用于向Selector注册Channel。
2. 提供EventLoopGroup中EventLoop线程的默认数量。

<font>register</font>

同样，交由EventExecutor实现，只是做委派。如下代码所示：

```java
@Override
public ChannelFuture register(Channel channel) {
    return next().register(channel);
}
@Override
public ChannelFuture register(ChannelPromise promise) {
    return next().register(promise);
}
@Deprecated
@Override
public ChannelFuture register(Channel channel, ChannelPromise promise) {
    return next().register(channel, promise);
}
```

<font>默认EventLoop数量</font>

```java
public abstract class MultithreadEventLoopGroup extends MultithreadEventExecutorGroup implements EventLoopGroup {
    ......
    private static final int DEFAULT_EVENT_LOOP_THREADS;
    static {
        DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
             "io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));
        ......
    }
	......
}
```

对于EventLoopGroup所提供的EventLoop线程的默认数量，是定义了一个常量`DEFAULT_EVENT_LOOP_THREADS`，通过静态代码块初始化，通过系统变量`io.netty.eventLoopThreads`获取，如果没有获取到，则调用`NettyRuntime.availableProcessors() * 2`获取，即CPU内核数量的2倍。

#### NioEventLoopGroup

NioEventLoopGroup完成了两件工作：

1. 在构造函数中提供默认的队列拒绝策略（RejectedExecutionHandlers）和seletor选择策略（DefaultSelectStrategyFactory）。
2. 实现MultithreadEventLoopGroup的newChild方法提供对EventLoop实例的创建。

<font>队列拒绝策略</font>

```java
public NioEventLoopGroup(int nThreads, ThreadFactory threadFactory,final SelectorProvider selectorProvider, final SelectStrategyFactory selectStrategyFactory) {
    super(nThreads, threadFactory, selectorProvider, selectStrategyFactory, RejectedExecutionHandlers.reject());
}
```

RejectedExecutionHandlers.reject()就是队列拒绝策略（参考？？）。

<font>Seletor选择策略</font>

```java
public NioEventLoopGroup(
    int nThreads, Executor executor, final SelectorProvider selectorProvider) {
    this(nThreads, executor, selectorProvider, DefaultSelectStrategyFactory.INSTANCE);
}
```

DefaultSelectStrategyFactory.INSTANCE就是seletor选择策略（参考？？）。

<font>EventLoop实例</font>

```java
@Override
protected EventLoop newChild(Executor executor, Object... args) throws Exception {
    SelectorProvider selectorProvider = (SelectorProvider) args[0];
    SelectStrategyFactory selectStrategyFactory = (SelectStrategyFactory) args[1];
    RejectedExecutionHandler rejectedExecutionHandler = (RejectedExecutionHandler) args[2];
    EventLoopTaskQueueFactory taskQueueFactory = null;
    EventLoopTaskQueueFactory tailTaskQueueFactory = null;

    int argsLength = args.length;
    if (argsLength > 3) {
        taskQueueFactory = (EventLoopTaskQueueFactory) args[3];
    }
    if (argsLength > 4) {
        tailTaskQueueFactory = (EventLoopTaskQueueFactory) args[4];
    }
    return new NioEventLoop(this, executor, selectorProvider,
                            selectStrategyFactory.newSelectStrategy(),
                            rejectedExecutionHandler, taskQueueFactory, tailTaskQueueFactory);
}
```

在NioEventLoopGroup中实现的newChild方法最后返回了一个NioEventLoop实例（它的类型就是EventExecutor）。

在创建NioEventLoop实例的时候，通过其构造函数传递了一些参数，除了executor之外，其他都是通过args参数获取：

- 第一个args参数：args[0]是SelectorProvider，其全类名是java.nio.channels.spi.SelectorProvider，它是原生NIO中的API，用于创建java.nio.channels.Selector。

- 第二个args参数：args[1]是SelectStrategyFactory，Seletor选择策略，如果NioEventLoopGroup的构造函数传递自定义的SelectStrategyFactory，那么就是默认提供的DefaultSelectStrategyFactory.INSTANCE。

- 第三个args参数：args[2]是RejectedExecutionHandler，队列拒绝策略，如果NioEventLoopGroup的构造函数传递自定义的RejectedExecutionHandler，那么就是默认提供的RejectedExecutionHandlers.reject()。
- 第四个args参数：args[4]是taskQueueFactory，用于创建任务队列的工厂。如果没有提供工厂，NioEventLoop会有自己内部的创建策略。
- 第五个args参数：args[5]是tailTaskQueueFactory，用于创建任务队列的工厂。？？如果没有提供工厂，NioEventLoop会有自己内部的创建策略。

### 总结

虽然NioEventLoopGroup提供了通道的注册操作，EventExecutor的生命周期管理（任务调度，线程关闭，线程选择），线程组的初始化功能，但实际上，真正由NioEventLoopGroup完成的工作是线程组的初始化。至于通道的注册操作，EventExecutor的生命周期管理等工作，都是交由具体的EventExecutor去执行。只是NioEventLoopGroup管理的是多个EventExecutor，而每个EventExecutor只能管理它自己，所以NioEventLoopGroup是对当前组中所有EventExecutor的管理，具体的操作则委派给对应的EventExecutor自己处理。

## NioEventLoop

NioEventLoop是基于TCP的事件循环。NioEventLoop具备多种职责，主要负责注册channel到selector，事件循环中多路复用，普通任务执以及延时任务执行四个功能。

NioEventLoop类继承关系如下：

![]()

```
EventLoop
OrderedEventExecutor
EventExecutor
EventExecutorGroup

java.util.concurrent.ExecutorService
java.util.concurrent.AbstractExecutorService
EventExecutorGroup EventExecutor AbstractEventExecutor
AbstractScheduledEventExecutor
EventExecutorGroup EventExecutor OrderedEventExecutor SingleThreadEventExecutor
EventLoop SingleThreadEventLoop
NioEventLoop
```



- `AbstractEventExecutor`

  主要实现了EventExecutor接口中定义的方法。

- `AbstractScheduledEventExecutor`

  负责定时任务的管理。

- `SingleThreadEventExecutor`

  负责普通任务的管理。

- `SingleThreadEventLoop`

  负责事件循环迭代结束时运行任务的管理，提供channel注册到selector的功能（委派）。

- `NioEventLoop`

  负责注册channel到selector，事件循环中多路复用。





### EventLoop

```java
public interface EventLoop extends OrderedEventExecutor, EventLoopGroup {
    @Override
    EventLoopGroup parent();
}
```

一旦注册，将处理通道的所有I/O操作。一个EventLoop实例通常会处理多个Channel，但这可能取决于实现细节和内部。

### EventExecutor 

EventExecutor是一个特殊的EventExecutor组，它带有一些方便的方法来查看一个线程是否在事件循环中执行。除此之外，它还扩展了EventExecutorGroup，以允许以通用的方式访问方法。

EventExecutor是一个特殊的EventExecutorGroup，它提供了一些方便的方法来查看一个线程是否在事件循环中执行。除此之外，它还扩展了EventExecutorGroup以允许以一种通用的方式访问方法。

```java
public interface EventExecutor extends EventExecutorGroup {
    @Override
    EventExecutor next();
    EventExecutorGroup parent();
    boolean inEventLoop();
    boolean inEventLoop(Thread thread);
    <V> Promise<V> newPromise();
    <V> ProgressivePromise<V> newProgressivePromise();
    <V> Future<V> newSucceededFuture(V result);
    <V> Future<V> newFailedFuture(Throwable cause);
}
```



- `EventExecutor next()`：返回对自身的引用。
- `EventExecutorGroup parent()`：返回EventExecutorGroup，它是这个EventExecutor的父节点，
- `boolean inEventLoop()`：使用Thread.currentThread()作为参数调用inEventLoop(Thread)
- `boolean inEventLoop(Thread thread)`：如果给定的线程在事件循环中执行，则返回true，否则返回false。
- `<V> Promise<V> newPromise()`：返回一个新的Promise。
- `<V> ProgressivePromise<V> newProgressivePromise()`：创建一个新的ProgressivePromise。
- `<V> Future<V> newSucceededFuture(V result)`：创建一个新的Future，它已经被标记为成功。所以Future.isSuccess()将返回true。所有添加到它的FutureListener都将被直接通知。而且，每次调用阻塞方法都会返回而不会阻塞。
- `<V> Future<V> newFailedFuture(Throwable cause)`：创建一个已经标记为失败的新Future。所以Future.isSuccess()会返回false。所有添加到它的FutureListener都将被直接通知。而且，每次调用阻塞方法都会返回而不会阻塞。





### OrderedEventExecutor

```java
public interface OrderedEventExecutor extends EventExecutor {}
```

 EventExecutors的标记接口，它将以有序/串行的方式处理所有提交的任务。



### AbstractEventExecutor

### AbstractScheduledEventExecutor

AbstractScheduledEventExecutor负责支持定时任务的管理。



```java
public abstract class AbstractScheduledEventExecutor extends AbstractEventExecutor {
    private static final Comparator<ScheduledFutureTask<?>> SCHEDULED_FUTURE_TASK_COMPARATOR =
            new Comparator<ScheduledFutureTask<?>>() {
                @Override
                public int compare(ScheduledFutureTask<?> o1, ScheduledFutureTask<?> o2) {
                    return o1.compareTo(o2);
                }
            };

   static final Runnable WAKEUP_TASK = new Runnable() {
       @Override
       public void run() { } // Do nothing
    };

    PriorityQueue<ScheduledFutureTask<?>> scheduledTaskQueue;

    long nextTaskId;
	......
}
```

·包含四个变量：

- `SCHEDULED_FUTURE_TASK_COMPARATOR`：
- `WAKEUP_TASK`：
- `scheduledTaskQueue`：定时任务优先队列，每一个定时任务都被封装为一个ScheduledFutureTask对象。
- `nextTaskId`：定时任务ID，为一个long型整数，从整数1开始。

针对定时任务优先队列的操作方法：

```java
protected static long nanoTime() {
    return ScheduledFutureTask.nanoTime();
}

protected static long deadlineToDelayNanos(long deadlineNanos) {
    return ScheduledFutureTask.deadlineToDelayNanos(deadlineNanos);
}

protected static long initialNanoTime() {
    return ScheduledFutureTask.initialNanoTime();
}

protected void cancelScheduledTasks() {
    assert inEventLoop();
    PriorityQueue<ScheduledFutureTask<?>> scheduledTaskQueue = this.scheduledTaskQueue;
    if (isNullOrEmpty(scheduledTaskQueue)) {
        return;
    }

    final ScheduledFutureTask<?>[] scheduledTasks =
        scheduledTaskQueue.toArray(new ScheduledFutureTask<?>[0]);

    for (ScheduledFutureTask<?> task: scheduledTasks) {
        task.cancelWithoutRemove(false);
    }

    scheduledTaskQueue.clearIgnoringIndexes();
}

protected final Runnable pollScheduledTask() {
    return pollScheduledTask(nanoTime());
}

protected final Runnable pollScheduledTask(long nanoTime) {
    assert inEventLoop();

    ScheduledFutureTask<?> scheduledTask = peekScheduledTask();
    if (scheduledTask == null || scheduledTask.deadlineNanos() - nanoTime > 0) {
        return null;
    }
    scheduledTaskQueue.remove();
    scheduledTask.setConsumed();
    return scheduledTask;
}

protected final long nextScheduledTaskNano() {
    ScheduledFutureTask<?> scheduledTask = peekScheduledTask();
    return scheduledTask != null ? scheduledTask.delayNanos() : -1;
}

protected final long nextScheduledTaskDeadlineNanos() {
    ScheduledFutureTask<?> scheduledTask = peekScheduledTask();
    return scheduledTask != null ? scheduledTask.deadlineNanos() : -1;
}

final ScheduledFutureTask<?> peekScheduledTask() {
    Queue<ScheduledFutureTask<?>> scheduledTaskQueue = this.scheduledTaskQueue;
    return scheduledTaskQueue != null ? scheduledTaskQueue.peek() : null;
}

protected final boolean hasScheduledTasks() {
    ScheduledFutureTask<?> scheduledTask = peekScheduledTask();
    return scheduledTask != null && scheduledTask.deadlineNanos() <= nanoTime();
}
```

函数功能描述如下：

- `nanoTime`：
- `deadlineToDelayNanos`：给定一个任意的截止日期，计算从现在起截止日期的纳秒数。
- `initialNanoTime`：用于基于单原子时间源的延迟和计算的初始值。
- `cancelScheduledTasks`：取消所有计划任务。只有当inEventLoop()为true时才必须调用此方法。
- `pollScheduledTask`：返回Runnable，它可以使用给定的nanoTime执行。您应该使用nanoTime()来检索正确的nanoTime。
- `nextScheduledTaskNano`：返回下一个计划任务准备运行之前的纳秒数，如果没有计划任务，则返回-1。
- `nextScheduledTaskDeadlineNanos`：当下一个定时任务准备运行时返回截止日期(以纳秒为单位)，如果没有计划任务，则返回-1。
- `peekScheduledTask`：
- `hasScheduledTasks`：如果计划任务可以进行处理，则返回true。

**调度任务**

调度任务分为两种：

1. schedule：
2. scheduleAtFixedRate：

```java
@Override
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) {
    ......
    return schedule(new ScheduledFutureTask<Void>(this, command,
                                                deadlineNanos(unit.toNanos(delay))));
}
@Override
public <V> ScheduledFuture<V> schedule(Callable<V> callable, 
                                       long delay, TimeUnit unit) {
    ......
    return schedule(new ScheduledFutureTask<V>(this, callable, 
                                                deadlineNanos(unit.toNanos(delay))));
}
@Override
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, 
                                     long initialDelay, long period, TimeUnit unit) {
    ......
    return schedule(new ScheduledFutureTask<Void>(
            this, command, deadlineNanos(unit.toNanos(initialDelay)), 		                     unit.toNanos(period)));
}
@Override
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, 
                                      long initialDelay, long delay, TimeUnit unit) {
    ......
    return schedule(new ScheduledFutureTask<Void>(
            this, command, deadlineNanos(unit.toNanos(initialDelay)), 
            -unit.toNanos(delay)));
}
```

四个调度任务函数实现都大同小异，都是将定时任务封装ScheduledFutureTask，然后调用#schedule(ScheduledFutureTask)方法：

```java
private <V> ScheduledFuture<V> schedule(final ScheduledFutureTask<V> task) {
    if (inEventLoop()) {
        scheduleFromEventLoop(task);
    } else {
        final long deadlineNanos = task.deadlineNanos();
        // 如果任务未过期，则在运行时将自己添加到计划任务队列中
        if (beforeScheduledTaskSubmitted(deadlineNanos)) {
            execute(task);
        } else {
            lazyExecute(task);
            // Second hook after scheduling to facilitate race-avoidance
            if (afterScheduledTaskSubmitted(deadlineNanos)) {
                execute(WAKEUP_TASK);
            }
        }
    }
    return task;
}

final void scheduleFromEventLoop(final ScheduledFutureTask<?> task) {
    // nextTaskId a long and so there is no chance it will overflow back to 0
    scheduledTaskQueue().add(task.setId(++nextTaskId));
}

PriorityQueue<ScheduledFutureTask<?>> scheduledTaskQueue() {
    if (scheduledTaskQueue == null) {
        scheduledTaskQueue = new DefaultPriorityQueue<ScheduledFutureTask<?>>(
            SCHEDULED_FUTURE_TASK_COMPARATOR,
            // Use same initial capacity as java.util.PriorityQueue
            11);
    }
    return scheduledTaskQueue;
}
```

首先调用inEventLoop()函数判断当前线程（Thread.currentThread()）是否在当前EventLoop中执行，由于我们使用schedule*()函数是一般是通过如下代码调用：

```java
ctx.channel().eventLoop().schedule(()->{...}, 10, TimeUnit.SECONDS);
```

这个调用位于ChannelHandler之下，因此当前线程(Thread.currentThread())必然在当前EventLoop中执行，即意味着这里inEventLoop()将返回true。然后调用scheduleFromEventLoop()将当前添加的定时任务设置任务ID并添加到定时任务优先队列中。最后schedule返回添加的定时任务。

另外注意到，获取定时任务优先队列是通过scheduledTaskQueue()方法，在scheduledTaskQueue()方法中，当首次添加定时任务时才会初始化定时任务优先队列，初始队列大小11。

什么情况下inEventLoop()才会返回false呢？当Thread.currentThread()不在在当前EventLoop中执行时，即调用schedule*()函数的线程不是当前EventLoop中的线程。例如：

```java
new Thread(()->{
    ctx.channel().eventLoop().schedule(()->{...}, 10, TimeUnit.SECONDS);
}).start();
```

新创建了一个线程调用schedule*()函数，此时inEventLoop()将返回false。

当inEventLoop()返回false时，首先获取当前定时任务的延时时间(ns)，然后调用beforeScheduledTaskSubmitted()函数，并将延时时间传递给它。

```java
protected boolean beforeScheduledTaskSubmitted(long deadlineNanos) {
    return true;
}
protected boolean afterScheduledTaskSubmitted(long deadlineNanos) {
    return true;
}
```

对于所有通过schedule调度的定时任务，无论是从EventExecutor线程调度还是通过非EventExecutor线程调度，都需要按照预定的定时时间调度，而不是需要什么特别的处理。目前定时任务调度所等待的时间是通过select()阻塞指定的时间实现的。对于从EventExecutor线程调度的定时任务来说，无论有多个定时任务，无论定时时间是多长，都会统一的添加到定时任务优先队列中，然后再都添加完成之后，ChannelHandler返回，然后环境select()，在从定时任务优先队列中取出堆中最小的任务进行阻塞。问题在于通过非EventExecutor线程调度的定时任务是并发添加的，而AbstractScheduledEventExecutor内部的优先队列scheduledTaskQueue是Netty自定义的一个实现，它并不是线程安全的。另一种情况是，如果有一个任务select()阻塞30s，而此时新添加的任务定时10s，那么久有必要唤醒select()使其重新阻塞为10s。

基于上述两种原因，通过非EventExecutor线程调度的定时任务不能直接添加到定时任务优先队列中。可是由于非EventExecutor线程的原因，我们不能确定任务会在什么时候被添加。一种简单的解决办法就是先将其添加到一个临时队列中，然后在运行时以串行的方式将其批量添加到定时任务优先队列中。

上述描述的方式就是Netty的实现策略。

beforeScheduledTaskSubmitted和afterScheduledTaskSubmitted的实现位于NioEventLoop中，实现如下所示。它们的实现是一样，以当前新添加的定时任务的调度时间与当前select()阻塞时间进行比较，如果调度时间小于阻塞时间，那么就需要将select()唤醒，因此返回true。

```java
@Override
protected boolean beforeScheduledTaskSubmitted(long deadlineNanos) {
    // Note this is also correct for the nextWakeupNanos == -1 (AWAKE) case
    return deadlineNanos < nextWakeupNanos.get();
}

@Override
protected boolean afterScheduledTaskSubmitted(long deadlineNanos) {
    // Note this is also correct for the nextWakeupNanos == -1 (AWAKE) case
    return deadlineNanos < nextWakeupNanos.get();
}
```

当beforeScheduledTaskSubmitted返回true是，调用execute，反之，调用lazyExecute。execute和lazyExecute在SingleThreadEventExecutor中实现的位于差别在于execute会唤醒select()，而lazyExecute不会。后续之所以会调用afterScheduledTaskSubmitted是为了避免冲突，因此当调用beforeScheduledTaskSubmitted的时间，select()并没有阻塞，而是出于允许状态，此时nextWakeupNanos.get()的返回值是-1，无论如何beforeScheduledTaskSubmitted将返回false。此时在调用lazyExecute将定时任务任务普通任务队列的时候，可能恰巧select()运行完毕，阻塞于一个延时时间大于当前定时任务的任务。所有有必要再次进行判断当前定时任务延时时间是否小于阻塞时间，以判断是否有必要进行唤醒。

### SingleThreadEventExecutor

SingleThreadEventExecutor负责在单个线程中执行所有任务，无论是普通任务还是定时任务。

```java
public abstract class SingleThreadEventExecutor extends AbstractScheduledEventExecutor implements OrderedEventExecutor {

    static final int DEFAULT_MAX_PENDING_EXECUTOR_TASKS = Math.max(16,
            SystemPropertyUtil.getInt("io.netty.eventexecutor.maxPendingTasks", Integer.MAX_VALUE));

    private static final InternalLogger logger =
            InternalLoggerFactory.getInstance(SingleThreadEventExecutor.class);

    private static final int ST_NOT_STARTED = 1;
    private static final int ST_STARTED = 2;
    private static final int ST_SHUTTING_DOWN = 3;
    private static final int ST_SHUTDOWN = 4;
    private static final int ST_TERMINATED = 5;

    private static final Runnable NOOP_TASK = new Runnable() {
        @Override
        public void run() {
            // Do nothing.
        }
    };

    private static final AtomicIntegerFieldUpdater<SingleThreadEventExecutor> STATE_UPDATER =
            AtomicIntegerFieldUpdater.newUpdater(SingleThreadEventExecutor.class, "state");
    private static final AtomicReferenceFieldUpdater<SingleThreadEventExecutor, ThreadProperties> PROPERTIES_UPDATER =
            AtomicReferenceFieldUpdater.newUpdater(
                    SingleThreadEventExecutor.class, ThreadProperties.class, "threadProperties");
	//普通任务队列
    private final Queue<Runnable> taskQueue;
	//当前EventExecutor线程
    private volatile Thread thread;
    @SuppressWarnings("unused")
    private volatile ThreadProperties threadProperties;
    private final Executor executor;
    private volatile boolean interrupted;

    private final CountDownLatch threadLock = new CountDownLatch(1);
    private final Set<Runnable> shutdownHooks = new LinkedHashSet<Runnable>();
    private final boolean addTaskWakesUp;
    private final int maxPendingTasks;
    private final RejectedExecutionHandler rejectedExecutionHandler;
	//
    private long lastExecutionTime;
    //事件循环状态
    @SuppressWarnings({ "FieldMayBeFinal", "unused" })
    private volatile int state = ST_NOT_STARTED;
    //优雅关机的静默时间
    private volatile long gracefulShutdownQuietPeriod;
    //优雅关机的超时时间
    private volatile long gracefulShutdownTimeout;
    //优雅关机的启动时间
    private long gracefulShutdownStartTime;

    private final Promise<?> terminationFuture = new DefaultPromise<Void>(GlobalEventExecutor.INSTANCE);
	......
}
```





- ST_NOT_STARTED：未启动
- ST_STARTED：已启动
- ST_SHUTTING_DOWN：
- ST_SHUTDOWN：
- ST_TERMINATED：



在SingleThreadEventExecutor中，核销的方式有两类，分别是runAllTasks和execute。runAllTasks用于运行所有任务，无论定时还是普通。execute用于调度普通任务。

#### execute

调度普通任务对外暴露的接口有两个，源码如下：

```java
@Override
public void execute(Runnable task) {
    ObjectUtil.checkNotNull(task, "task");
    execute(task, !(task instanceof LazyRunnable) && wakesUpForTask(task));
}

@Override
public void lazyExecute(Runnable task) {
    execute(ObjectUtil.checkNotNull(task, "task"), false);
}
```

它们的差异在于第二个参数，这个参数的作用是用于判断是否唤醒select()，true表示唤醒。对于execute而言，如果给定的任务类型不是LazyRunnable，将唤醒select()。至于wakesUpForTask()函数，它没有实现，固定返回true。

wakesUpForTask

```java
protected boolean wakesUpForTask(Runnable task) {
    return true;
}
```

然后调用execute(Runnable, boolean)：

```java
private void execute(Runnable task, boolean immediate) {
    boolean inEventLoop = inEventLoop();
    addTask(task);
    if (!inEventLoop) {
        startThread();
        if (isShutdown()) {
            boolean reject = false;
            try {
                if (removeTask(task)) {
                    reject = true;
                }
            } catch (UnsupportedOperationException e) {
                // 任务队列不支持删除，所以我们能做的最好的事情就是继续前进，
                // 希望我们能够在任务完全终止之前将其取出。
                // In worst case we will log on termination.
                // 在最坏的情况下，我们将登录终止。
            }
            if (reject) {
                reject();
            }
        }
    }

    if (!addTaskWakesUp && immediate) {
        wakeup(inEventLoop);
    }
}
```

首先是获取inEventLoop()，即存在EventExecutor线程和非EventExecutor线程调度的问题。然后将任务添加到普通任务队列中，最后再根据判断是否需要唤醒select()。在这之间，如果是非EventExecutor线程调度，那么将调用startThread()启动循环（当然，如果已经启动了肯定就不会再启动）。然后再判断一下事件循环是否已经终止，如果终止，则将任务从普通任务队列中删除，然后执行拒绝策略，即抛出RejectedExecutionException异常。

```java
protected static void reject() {
    throw new RejectedExecutionException("event executor terminated");
}
```

---

startThread()启动循环

```java
private void startThread() {
    if (state == ST_NOT_STARTED) {
        if (STATE_UPDATER.compareAndSet(this, ST_NOT_STARTED, ST_STARTED)) {
            boolean success = false;
            try {
                doStartThread();
                success = true;
            } finally {
                if (!success) {
                    STATE_UPDATER.compareAndSet(this, ST_STARTED, ST_NOT_STARTED);
                }
            }
        }
    }
}
```

如果时间循环状态是ST_NOT_STARTED，则通过CAS将其变更为ST_STARTED，然后调用doStartThread()执行真正的启动操作。如果启动失败，则将状态又变更为ST_NOT_STARTED。

```java
private void doStartThread() {
    assert thread == null;
    executor.execute(new Runnable() {
        @Override
        public void run() {
            //赋值当前EventExecutor线程
            thread = Thread.currentThread();
            if (interrupted) {
                thread.interrupt();
            }

            boolean success = false;
            //更新定时任务的最后执行时间
            updateLastExecutionTime();
            try {
                //事件循环
                SingleThreadEventExecutor.this.run();
                success = true;
            } catch (Throwable t) {
                logger.warn("Unexpected exception from an event executor: ", t);
            } finally {
                //将状态标记为ST_SHUTTING_DOWN
                for (;;) {
                    int oldState = state;
                    if (oldState >= ST_SHUTTING_DOWN || STATE_UPDATER.compareAndSet(
                        SingleThreadEventExecutor.this, oldState, ST_SHUTTING_DOWN)) {
                        break;
                    }
                }

                // 检查是否在循环结束时调用了confirmShutdown()。
                if (success && gracefulShutdownStartTime == 0) {
                    if (logger.isErrorEnabled()) {
                        logger.error("Buggy " + EventExecutor.class.getSimpleName() + " implementation; " +
                                     SingleThreadEventExecutor.class.getSimpleName() + ".confirmShutdown() must " +
                                     "be called before run() implementation terminates.");
                    }
                }

                try {
                    // 运行所有剩余的任务并关闭挂钩。此时事件循环处于ST_SHUTTING_DOWN状态，
                    // 仍然接受使用quietPeriod优雅关闭所需的任务。
                    for (;;) {
                        if (confirmShutdown()) {
                            break;
                        }
                    }

                    // 现在我们要确保从这一点开始不再添加任何任务。这是通过切换状态实现的。
                    // 超过这个时间点的任何新任务都将被拒绝。
                    for (;;) {
                        int oldState = state;
                        if (oldState >= ST_SHUTDOWN || STATE_UPDATER.compareAndSet(
                            SingleThreadEventExecutor.this, oldState, ST_SHUTDOWN)) {
                            break;
                        }
                    }

                    // 现在队列中有了最后一组任务，不能再添加了，运行所有剩余的任务。
                    // 这里不需要循环，这是最后一关。
                    confirmShutdown();
                } finally {
                    try {
                        cleanup();
                    } finally {
                        // 让我们移除该线程的所有FastThreadLocals，
                        // 因为我们即将终止并通知future。用户可能会在将来阻塞，
                        // 一旦解除阻塞，JVM可能会终止并开始卸载类。
                        // See https://github.com/netty/netty/issues/6596.
                        FastThreadLocal.removeAll();

                        STATE_UPDATER.set(SingleThreadEventExecutor.this, ST_TERMINATED);
                        threadLock.countDown();
                        int numUserTasks = drainTasks();
                        if (numUserTasks > 0 && logger.isWarnEnabled()) {
                            logger.warn("An event executor terminated with " +
                                        "non-empty task queue (" + numUserTasks + ')');
                        }
                        terminationFuture.setSuccess(null);
                    }
                }
            }
        }
    });
}
```







```java
protected boolean confirmShutdown() {
    if (!isShuttingDown()) {
        return false;
    }

    if (!inEventLoop()) {
        throw new IllegalStateException("must be invoked from an event loop");
    }

    cancelScheduledTasks();
	//设置优雅关机的启动时间
    if (gracefulShutdownStartTime == 0) {
        gracefulShutdownStartTime = ScheduledFutureTask.nanoTime();
    }
	//允许所有任务和关闭钩子方法
    if (runAllTasks() || runShutdownHooks()) {
        // 已经关闭，返回true
        if (isShutdown()) {
            // Executor关闭-不再有新的任务。
            return true;
        }

        // There were tasks in the queue. Wait a little bit more until no tasks are queued for the quiet period or terminate if the quiet period is 0.
        // 队列中有任务。再等待一段时间，直到安静期没有任务排队，如果安静期为0则终止。
        // See https://github.com/netty/netty/issues/4241
        if (gracefulShutdownQuietPeriod == 0) {
            return true;
        }
        taskQueue.offer(WAKEUP_TASK);
        return false;
    }

    final long nanoTime = ScheduledFutureTask.nanoTime();
	//已经关闭或者优雅关机已经超时，返回true
    if (isShutdown() || nanoTime - gracefulShutdownStartTime > gracefulShutdownTimeout) {
        return true;
    }

    if (nanoTime - lastExecutionTime <= gracefulShutdownQuietPeriod) {
        // 每100毫秒检查是否有任务被添加到队列中。
        // TODO: 更改takeTask()的行为，使其在超时时返回。
        taskQueue.offer(WAKEUP_TASK);
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            // Ignore
        }

        return false;
    }

    // No tasks were added for last quiet period - hopefully safe to shut down.
    // 最后一个安静期没有添加任务-希望可以安全关闭。
    // (希望如此，因为我们确实不能保证用户不会调用execute()。)
    return true;
}
```

#### runAllTask

runAllTask

```java
protected boolean runAllTasks(long timeoutNanos) {
    fetchFromScheduledTaskQueue();
    //从普通队列中拉取一个任务
    Runnable task = pollTask();
    if (task == null) {
        //执行运行后任务
        afterRunningAllTasks();
        return false;
    }
	// 本次运行任务的超时时间
    final long deadline = timeoutNanos > 0 ? ScheduledFutureTask.nanoTime() + timeoutNanos : 0;
    long runTasks = 0;
    long lastExecutionTime;
    for (;;) {
        //执行任务
        safeExecute(task);
        //统计执行的任务个数
        runTasks ++; 
        // 每64个任务检查一次超时，因为nanoTime()的开销相对较大。
        if ((runTasks & 0x3F) == 0) {
            lastExecutionTime = ScheduledFutureTask.nanoTime();
            //最后一次执行时间大于等于本次运行任务的超时时间，则退出不在执行
            if (lastExecutionTime >= deadline) {
                break;
            }
        }
		
        task = pollTask();
        if (task == null) {
            lastExecutionTime = ScheduledFutureTask.nanoTime();
            break;
        }
    }
	//执行运行后任务
    afterRunningAllTasks();
    this.lastExecutionTime = lastExecutionTime;
    return true;
}
```

runAllTasks(long)有一个参数timeoutNanos，这个参数限制的是本次执行任务的时长，这是为了避免队列中任务过多导致长时间阻塞。由于nanoTime()的开销相对较大，所以会没允许64个任务检查一次是否超时。整体而言，runAllTasks(long)首先从定时任务队列中拉取可以执行的任务，然后将其放入普通任务队列中统一轮询执行。最多执行timeoutNanos纳秒时长。执行完成之后，执行运行后任务。

fetchFromScheduledTaskQueue

```java
private boolean fetchFromScheduledTaskQueue() {
    if (scheduledTaskQueue == null || scheduledTaskQueue.isEmpty()) {
        return true;
    }
    long nanoTime = AbstractScheduledEventExecutor.nanoTime();
    for (;;) {
        //从定时任务队列中拉取队列头部且未过期的定时任务
        Runnable scheduledTask = pollScheduledTask(nanoTime);
        if (scheduledTask == null) {
            return true;
        }
        //定时任务插入普通任务队列中
        if (!taskQueue.offer(scheduledTask)) {
            // 任务队列中没有剩余的空间了，把它添加到scheduledTaskQueue中，
            // 这样我们就可以再次获取它。
            scheduledTaskQueue.add((ScheduledFutureTask<?>) scheduledTask);
            return false;
        }
    }
}
```

fetchFromScheduledTaskQueue的目的就是从定时任务队列头部获取已经已过期（已经到达定时执行时间）的定时任务，然后将其放入普通任务队列中。

与之相对的是另一个runAllTasks()，它没有timeoutNanos参数：

```java
protected boolean runAllTasks() {
    assert inEventLoop();
    boolean fetchedAll;
    boolean ranAtLeastOne = false;

    do {
        fetchedAll = fetchFromScheduledTaskQueue();
        if (runAllTasksFrom(taskQueue)) {
            ranAtLeastOne = true;
        }
    } while (!fetchedAll); // keep on processing until we fetched all scheduled tasks.

    if (ranAtLeastOne) {
        lastExecutionTime = ScheduledFutureTask.nanoTime();
    }
    afterRunningAllTasks();
    return ranAtLeastOne;
}

protected final boolean runAllTasksFrom(Queue<Runnable> taskQueue) {
    Runnable task = pollTaskFrom(taskQueue);
    if (task == null) {
        return false;
    }
    for (;;) {
        safeExecute(task);
        task = pollTaskFrom(taskQueue);
        if (task == null) {
            return true;
        }
    }
}
```

与runAllTasks(long)大同小异，不同的是，runAllTasks()会执行完队列中的所有可执行任务，而不是有指定的时间长度限制。





### SingleThreadEventLoop

EventLoops的抽象基类，它在单个线程中执行所有提交的任务。

### NioEventLoop

NioEventLoop负责注册channel到selector和事件循环中多路复用。



```java
public final class NioEventLoop extends SingleThreadEventLoop {
	......
    private static final int CLEANUP_INTERVAL = 256; // XXX Hard-coded value, but won't need customization.

    private static final boolean DISABLE_KEY_SET_OPTIMIZATION =
            SystemPropertyUtil.getBoolean("io.netty.noKeySetOptimization", false);
	// selector自动重建最小阈值
    private static final int MIN_PREMATURE_SELECTOR_RETURNS = 3;
    // selector自动重建阈值
    private static final int SELECTOR_AUTO_REBUILD_THRESHOLD;
	// 带select结果的Supplier，用于运行时，select策略的第一个参数
    private final IntSupplier selectNowSupplier = new IntSupplier() {
        @Override
        public int get() throws Exception {
            return selectNow();
        }
    };

    // Workaround for JDK NIO bug.
    //
    // See:
    // - https://bugs.java.com/view_bug.do?bug_id=6427854
    // - https://github.com/netty/netty/issues/203
    static {
        final String key = "sun.nio.ch.bugLevel";
        final String bugLevel = SystemPropertyUtil.get(key);
        if (bugLevel == null) {
            try {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    @Override
                    public Void run() {
                        System.setProperty(key, "");
                        return null;
                    }
                });
            } catch (final SecurityException e) {
                logger.debug("Unable to get/set System Property: " + key, e);
            }
        }

        int selectorAutoRebuildThreshold = SystemPropertyUtil.getInt("io.netty.selectorAutoRebuildThreshold", 512);
        if (selectorAutoRebuildThreshold < MIN_PREMATURE_SELECTOR_RETURNS) {
            selectorAutoRebuildThreshold = 0;
        }

        SELECTOR_AUTO_REBUILD_THRESHOLD = selectorAutoRebuildThreshold;

        if (logger.isDebugEnabled()) {
            logger.debug("-Dio.netty.noKeySetOptimization: {}", DISABLE_KEY_SET_OPTIMIZATION);
            logger.debug("-Dio.netty.selectorAutoRebuildThreshold: {}", SELECTOR_AUTO_REBUILD_THRESHOLD);
        }
    }

    /**
     * The NIO {@link Selector}.
     */
    private Selector selector;
    private Selector unwrappedSelector;
    private SelectedSelectionKeySet selectedKeys;
	//JDK原型SelectorProvider，用于创建Selector
    private final SelectorProvider provider;
    // 事件循环状态 → 运行中
    private static final long AWAKE = -1L;
    // 事件循环状态 → 永久阻塞中
    private static final long NONE = Long.MAX_VALUE;
    // 事件循环状态
    // nextWakeupNanos is:
    //    AWAKE            when EL is awake
    //    NONE             when EL is waiting with no wakeup scheduled
    //    other value T    when EL is waiting with wakeup scheduled at time T
    private final AtomicLong nextWakeupNanos = new AtomicLong(AWAKE);
	//select策略，默认类型为DefaultSelectStrategy
    private final SelectStrategy selectStrategy;
	//I/O事件和Task执行比率
    private volatile int ioRatio = 50;
    private int cancelledKeys;
    private boolean needsToSelectAgain;
	......
}
```



<font>事件循环状态</font>

事件循环的状态通过类型为AtomicLong的变量nextWakeupNanos标记，并有两个相关状态常量:

```java
private static final long AWAKE = -1L;
private static final long NONE = Long.MAX_VALUE;
private final AtomicLong nextWakeupNanos = new AtomicLong(AWAKE);
```

状态有三个，分别是

1. AWAKE：运行中。
2. NONE：select()永久阻塞。
3. other value T：select(timeoutMillis)有限阻塞。

这三个状态在时间循环中进行更新以及在调用wake()进行唤醒时更新。主要是用于计划任务的添加控制。



#### Select策略

事件循环中用到了Select策略，Select策略在NioEventLoopGroup的构造函数中设置，如果没有指定自定义的Select策略，那么默认的Select策略通过DefaultSelectStrategyFactory#newSelectStrategy()工厂方法创建：

```java
public final class DefaultSelectStrategyFactory implements SelectStrategyFactory {
    public static final SelectStrategyFactory INSTANCE = new DefaultSelectStrategyFactory();
    private DefaultSelectStrategyFactory() { }
    @Override
    public SelectStrategy newSelectStrategy() {
        return DefaultSelectStrategy.INSTANCE;
    }
}
```

Select策略接口定义如下：

```java
public interface SelectStrategy {
    int SELECT = -1;
    int CONTINUE = -2;
    int BUSY_WAIT = -3;
    int calculateStrategy(IntSupplier selectSupplier, boolean hasTasks) throws Exception;
}
```

其作用是提供控制select循环的能力。例如，如果有事件需要立即处理，阻塞select操作可以被延迟或跳过。其提供了三种select策略：

- SELECT：指示后面应该有一个阻塞选择。
- CONTINUE：指示应重试IO循环，不直接进行阻塞选择。
- BUSY_WAIT：表示IO循环，以轮询新的事件而不阻塞。

calculateStrategy()函数用来控制可能的seelct调用的结果。如果下一步应该阻塞，则select，如果下一步不应该select，而是跳回IO循环并再次尝试，则选择CONTINUE。任何值>= 0都被视为需要完成工作的指示符。

对应的策略实现是DefaultSelectStrategy，实现很简单，就是通过hasTasks参数以判断是返回selectSupplier还是SELECT策略。

```java
final class DefaultSelectStrategy implements SelectStrategy {
    static final SelectStrategy INSTANCE = new DefaultSelectStrategy();
    private DefaultSelectStrategy() { }
    @Override
    public int calculateStrategy(IntSupplier selectSupplier, boolean hasTasks) throws Exception {
        return hasTasks ? selectSupplier.get() : SelectStrategy.SELECT;
    }
}
```

#### 事件循环

事件循环多路复用才做是由实现SingleThreadEventExecutor的抽象run()方法实现，源码如下：

```java
@Override
protected void run() {
    int selectCnt = 0;
    for (;;) {
        try {
            int strategy;
            try {
                strategy = selectStrategy.calculateStrategy(selectNowSupplier, hasTasks());
                switch (strategy) {
                    case SelectStrategy.CONTINUE:
                        continue;

                    case SelectStrategy.BUSY_WAIT:
                        // 由于NIO不支持繁忙等待，所以切换到SELECT
                    case SelectStrategy.SELECT:
                        long curDeadlineNanos = nextScheduledTaskDeadlineNanos();
                        if (curDeadlineNanos == -1L) {
                            curDeadlineNanos = NONE; // nothing on the calendar
                        }
                        nextWakeupNanos.set(curDeadlineNanos);
                        try {
                            if (!hasTasks()) {
                                strategy = select(curDeadlineNanos);
                            }
                        } finally {
                            // This update is just to help block unnecessary selector wakeups so use of lazySet is ok (no race condition)
                            // 
                            nextWakeupNanos.lazySet(AWAKE);
                        }
                        // fall through
                    default:
                }
            } catch (IOException e) {
                // If we receive an IOException here its because the Selector is messed up. Let's rebuild the selector and retry. 
                // https://github.com/netty/netty/issues/8566
                rebuildSelector0();
                selectCnt = 0;
                handleLoopException(e);
                continue;
            }

            selectCnt++;
            cancelledKeys = 0;
            needsToSelectAgain = false;
            final int ioRatio = this.ioRatio;
            boolean ranTasks;
            if (ioRatio == 100) {
                try {
                    // strategy大于0，表示有监听I/O事件，需要执行I/O事件处理
                    if (strategy > 0) {
                        processSelectedKeys();
                    }
                } finally {
                    // 此处I/O事件和Task的比率为100，因此执行队列中的所有任务，
                    // 直到执行完毕为止
                    ranTasks = runAllTasks();
                }
            } else if (strategy > 0) {
                final long ioStartTime = System.nanoTime();
                try {
                    //strategy大于0，有监听I/O事件，执行I/O事件处理
                    processSelectedKeys();
                } finally {
                    // Ensure we always run tasks.
                    final long ioTime = System.nanoTime() - ioStartTime;
                    // 运行所有任务，并按百分比计算运行任务限制的截止时间
                    ranTasks = runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
                }
            } else {
                // 此时strategy可能等于0（没有I/O事件）或-1（阻塞前添加了任务）。
                // 这将运行最小数量的任务，即默认最多64个任务
                ranTasks = runAllTasks(0);
            }
			// 有任务被运行，或有I/O事件被处理
            if (ranTasks || strategy > 0) {
                if (selectCnt > MIN_PREMATURE_SELECTOR_RETURNS && logger.isDebugEnabled()) {
                    logger.debug("Selector.select() returned prematurely {} times in a row for Selector {}.",
                                 selectCnt - 1, selector);
                }
                selectCnt = 0;
            } else if (unexpectedSelectorWakeup(selectCnt)) { // Unexpected wakeup (unusual case)
                selectCnt = 0;
            }
        } catch (CancelledKeyException e) {
            // Harmless exception - log anyway
            if (logger.isDebugEnabled()) {
                logger.debug(CancelledKeyException.class.getSimpleName() + " raised by a Selector {} - JDK bug?",
                             selector, e);
            }
        } catch (Error e) {
            throw e;
        } catch (Throwable t) {
            handleLoopException(t);
        } finally {
            // Always handle shutdown even if the loop processing threw an exception.
            try {
                if (isShuttingDown()) {
                    closeAll();
                    if (confirmShutdown()) {
                        return;
                    }
                }
            } catch (Error e) {
                throw e;
            } catch (Throwable t) {
                handleLoopException(t);
            }
        }
    }
}
```

首先是调用SelectStrategy#calculateStrategy获取一个select策略，传递了两个参数selectNowSupplier和hasTasks()，hasTasks()用于判断任务队列（普通和尾部）中是否存在任务。由DefaultSelectStrategy的实现可知，当队列中不存在任务时，将返回SELECT(-1)策略。如果存在将返回selectNowSupplier的值。

selectNowSupplier是NioEventLoop内部定义的一个成员变量，最终它调用了selectNow()。如下所示：

```java
private final IntSupplier selectNowSupplier = new IntSupplier() {
    @Override
    public int get() throws Exception {
        return selectNow();
    }
};
int selectNow() throws IOException {
    return selector.selectNow();
}
```

这意味着selectNowSupplier一定大于等于0，因此，当队列中存在任务时，将不会匹配任何select策略。

当select策略为SELECT(-1)，将调用select()阻塞，

1. 获取下一个计划任务的调度时间(纳秒)，如果不存在计划任务，则curDeadlineNanos为-1；

2. 如果curDeadlineNanos为-1，则将curDeadlineNanos设为NONE(Long.MAX_VALUE)。

3. 然后将nextWakeupNanos设置为curDeadlineNanos，即标记下一个计划任务需要的等待时间，以用于计划任务的调度判断（参见AbstractScheduledEventExecutor小节）。

4. 下一步就是判断任务队列中是否存在任务，当不存在时，调用select()多路复用。其实进入到SELECT(-1)策略，就意味着任务队列中没有任务，之所以再次进行判断，是因为可能由于并发的原因，在阻塞之间，此时恰巧有任务被添加。所以要进行再一次判断，存在任务就阻塞，去执行它。

5. 调用select()阻塞，如果deadlineNanos等于NONE，则就用阻塞；如果调度时间以过期或在5微秒内过期，则timeoutMillis将为0，及调用selectNow()不阻塞；反之，则阻塞timeoutMillis毫秒。

   ```java
   private int select(long deadlineNanos) throws IOException {
       if (deadlineNanos == NONE) {
           return selector.select();
       }
       // 如果调度时间在5微秒内截止，timeoutMillis将为0
       long timeoutMillis = deadlineToDelayNanos(deadlineNanos + 995000L) / 1000000L;
       return timeoutMillis <= 0 ? selector.selectNow() : selector.select(timeoutMillis);
   }
   ```

6. 可能sleect()被唤醒，也可能存在任务没有阻塞，不管如何，到此处select处理允许状态，将将nextWakeupNanos设置为AWAKE(-1)，及表示sleect处于运行状态。

下一阶段是处理I/O时间和运行任务。由于NioEventLoop需要同时处理I/O事件和非I/O任务，为了保证两者都得到足够的时间被执行，Netty提供了I/O事件和非I/O任务的执行比率的定制。定制操作位于NioEventLoop#setIoRatio()和NioEventLoopGroup#setIoRatio()。

NioEventLoop

```java
public void setIoRatio(int ioRatio) {
    if (ioRatio <= 0 || ioRatio > 100) {
        throw new IllegalArgumentException("ioRatio: " + ioRatio + " (expected: 0 < ioRatio <= 100)");
    }
    this.ioRatio = ioRatio;
}
```

NioEventLoopGroup

```java
public void setIoRatio(int ioRatio) {
    for (EventExecutor e: this) {
        ((NioEventLoop) e).setIoRatio(ioRatio);
    }
}
```

如果I/O操作多余定时任务和Task，则可以将I/O比率调大，反之调小。默认·比率为50%。

在我们没有设置执行比率的情况下，if判断的第一个分支永远不会进入。在这三个分支中，全两个分支都是在有I/O事件的情况下先执行I/O事件，然后执行任务。不同的是第一个分支执行所有的I/O事件（如果有）和任务。第二个分支按比率执行。第三个分支只执行最小数量(64)的任务，这个分支中一定没有I/O事件。

#### JDK NIO bug



#### selector重建

selector重建通过unexpectedSelectorWakeup()方法，

```java
// 如果selectCnt需要重置，则返回true
private boolean unexpectedSelectorWakeup(int selectCnt) {
    if (Thread.interrupted()) {
        // 线程被中断，所以重置选定的键并中断，这样我们就不会进入一个繁忙的循环。
        // 因为这很可能是用户处理程序或它的客户端库中的一个错误，我们也将记录它。
        // See https://github.com/netty/netty/issues/2426
        if (logger.isDebugEnabled()) {
            logger.debug("Selector.select() returned prematurely because " +
               "Thread.currentThread().interrupt() was called. Use " +
               "NioEventLoop.shutdownGracefully() to shutdown the NioEventLoop.");
        }
        return true;
    }
    if (SELECTOR_AUTO_REBUILD_THRESHOLD > 0 &&
        selectCnt >= SELECTOR_AUTO_REBUILD_THRESHOLD) {
        // selector在一行中多次提前返回。
        // 重新构建selector以解决该问题。
        logger.warn("Selector.select() returned prematurely {} times in a row; rebuilding Selector {}.", selectCnt, selector);
        rebuildSelector();
        return true;
    }
    return false;
}
```

如果线程已经被中断，自然没有重建的必要，直接返回true。重建的前提条件是重建阈值大于0并且事件循环次数大于等于阈值。当满足条件时，调用rebuildSelector()重建。

```java
/**
     * Replaces the current {@link Selector} of this event loop with newly created {@link Selector}s to work
     * around the infamous epoll 100% CPU bug.
     */
public void rebuildSelector() {
    if (!inEventLoop()) {
        execute(new Runnable() {
            @Override
            public void run() {
                rebuildSelector0();
            }
        });
        return;
    }
    rebuildSelector0();
}
```

重建仍然以普通任务的形式执行，最终调用rebuildSelector0()：

```java
private void rebuildSelector0() {
    final Selector oldSelector = selector;
    final SelectorTuple newSelectorTuple;
    if (oldSelector == null) {
        return;
    }
    try {
        // 创建新的Selector
        newSelectorTuple = openSelector();
    } catch (Exception e) {
        logger.warn("Failed to create a new Selector.", e);
        return;
    }
    // 注册所有通道到新的Selector。
    int nChannels = 0;
    // 迭代旧Selector上的所有通道
    for (SelectionKey key: oldSelector.keys()) {
        Object a = key.attachment();
        try {
            // 通道无效或已经注册到新的Selector上，则跳过
            if (!key.isValid() || key.channel().keyFor(newSelectorTuple.unwrappedSelector) != null) {
                continue;
            }
			
            int interestOps = key.interestOps();
            key.cancel();
            // 注册
            SelectionKey newKey = key.channel().register(newSelectorTuple.unwrappedSelector, interestOps, a);
            if (a instanceof AbstractNioChannel) {
                // Update SelectionKey
                ((AbstractNioChannel) a).selectionKey = newKey;
            }
            nChannels ++;
        } catch (Exception e) {
            logger.warn("Failed to re-register a Channel to the new Selector.", e);
            if (a instanceof AbstractNioChannel) {
                AbstractNioChannel ch = (AbstractNioChannel) a;
                ch.unsafe().close(ch.unsafe().voidPromise());
            } else {
                @SuppressWarnings("unchecked")
                NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
                invokeChannelUnregistered(task, key, e);
            }
        }
    }
	// 更新Selector
    selector = newSelectorTuple.selector;
    unwrappedSelector = newSelectorTuple.unwrappedSelector;
    try {
        // 关闭旧的Selector，因为此时所有其他的都注册到新的Selector
        oldSelector.close();
    } catch (Throwable t) {
        if (logger.isWarnEnabled()) {
            logger.warn("Failed to close the old Selector.", t);
        }
    }
    if (logger.isInfoEnabled()) {
        logger.info("Migrated " + nChannels + " channel(s) to the new Selector.");
    }
}
```

重建操作很简单，就是先创建一个新的Selector，然后将旧的Selector上的所有注册到新的Selector上，最后关闭旧Selector。如此，就重建完毕了。

<font>重建阈值初始化</font>

前面介绍到Selector的重建有一个阈值，这个在NioEventLoop的静态代码块中完成初始化加载：

```java
// Workaround for JDK NIO bug.
//
// See:
// - https://bugs.java.com/view_bug.do?bug_id=6427854
// - https://github.com/netty/netty/issues/203
static {
    // 如果NIO的BUG级别为NULL，则将其置为""
    final String key = "sun.nio.ch.bugLevel";
    final String bugLevel = SystemPropertyUtil.get(key);
    if (bugLevel == null) {
        try {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                @Override
                public Void run() {
                    System.setProperty(key, "");
                    return null;
                }
            });
        } catch (final SecurityException e) {
            logger.debug("Unable to get/set System Property: " + key, e);
        }
    }
    // 重系统变量中获取重建阈值，如果重建阈值小于3，则将重建阈值将置为0，
    int selectorAutoRebuildThreshold = SystemPropertyUtil.getInt("io.netty.selectorAutoRebuildThreshold", 512);
    if (selectorAutoRebuildThreshold < MIN_PREMATURE_SELECTOR_RETURNS) {
        selectorAutoRebuildThreshold = 0;
    }

    SELECTOR_AUTO_REBUILD_THRESHOLD = selectorAutoRebuildThreshold;

    if (logger.isDebugEnabled()) {
        logger.debug("-Dio.netty.noKeySetOptimization: {}", DISABLE_KEY_SET_OPTIMIZATION);
        logger.debug("-Dio.netty.selectorAutoRebuildThreshold: {}", SELECTOR_AUTO_REBUILD_THRESHOLD);
    }
}
```

执行了两步操作：

1. 如果没有设置NIO的BUG级别，则将BUG级别设置为`""`。其目的是为了解决JDK NIO bug。
2. 从系统变量中获取重建阈值，重建阈值不能小于3，否则重建阈值将置为0，将导致Selector不会重建。默认情况下，重建阈值为512。

#### 处理I/O事件

当事件循环轮询到I/O事件时，就需要处理网络I/O事件。源码如下：

```java
private void processSelectedKeys() {
    if (selectedKeys != null) {
        processSelectedKeysOptimized();
    } else {
        processSelectedKeysPlain(selector.selectedKeys());
    }
}
```

由于未开启selectedKeys的优化功能，因此会进入processSelectedKeysPlain()分支执行。

```java
private void processSelectedKeysPlain(Set<SelectionKey> selectedKeys) {
    // 检查集合是否为空，如果为空，即使没有任何东西需要处理，
    // 也要通过每次创建一个新的Iterator返回以避免创建垃圾。
    // See https://github.com/netty/netty/issues/597
    if (selectedKeys.isEmpty()) {
        return;
    }

    Iterator<SelectionKey> i = selectedKeys.iterator();
    for (;;) {
        final SelectionKey k = i.next();
        final Object a = k.attachment();
        i.remove();

        if (a instanceof AbstractNioChannel) {
            // Netty创建的channel
            processSelectedKey(k, (AbstractNioChannel) a);
        } else {
            // NioEventLoop#register()注册的channel
            @SuppressWarnings("unchecked")
            NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
            processSelectedKey(k, task);
        }
		// 判断是否还有未被处理的I/O事件，没有则退出
        if (!i.hasNext()) {
            break;
        }

        if (needsToSelectAgain) {
            selectAgain();
            selectedKeys = selector.selectedKeys();

            // 再次创建迭代器以避免ConcurrentModificationException
            if (selectedKeys.isEmpty()) {
                break;
            } else {
                i = selectedKeys.iterator();
            }
        }
    }
}

private void selectAgain() {
    needsToSelectAgain = false;
    try {
        selector.selectNow();
    } catch (Throwable t) {
        logger.warn("Failed to update SelectionKeys.", t);
    }
}
```

通过给定的SelectionKey集合，迭代每一个SelectionKey，根据其附加对象，有两个分支，一个是附加对象类型为AbstractNioChannel，则表示当前channel是由Netty创建的。反之，则表示当前channel是通过NioEventLoop#register()注册的。进一步继续交由processSelectedKey(?)函数处理。最后判断是否需要再次select，如果需要，则继续轮询。反之则退出循环，I/O事件处理完毕。

<font>channel由Netty创建</font>

如果channel由Netty创建，则交由processSelectedKey(SelectionKey,AbstractNioChannel)处理，源码如下：

```java
private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
    if (!k.isValid()) {
        final EventLoop eventLoop;
        try {
            eventLoop = ch.eventLoop();
        } catch (Throwable ignored) {
            // 如果通道实现因为没有事件循环而抛出异常，则忽略该异常，因
            // 为我们只是试图确定ch是否注册到该事件循环，从而有权关闭ch。
            return;
        }
        // 只有当ch仍然注册到这个EventLoop时才关闭ch。ch可能已经从事件循环中注销了，
        // 因此可以作为注销过程的一部分取消SelectionKey，但通道仍然正常，不应关闭。
        // See https://github.com/netty/netty/issues/5125
        if (eventLoop == this) {
            // close the channel if the key is not valid anymore
            unsafe.close(unsafe.voidPromise());
        }
        return;
    }

    try {
        int readyOps = k.readyOps();
        // 在尝试触发read(…)或write(…)之前，我们首先需要调用finishConnect()，
        // 否则NIO JDK通道实现可能会抛出NotYetConnectedException异常。
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            // 移除OP_CONNECT，否则Selector.select(..)将始终返回而不会阻塞
            // See https://github.com/netty/netty/issues/924
            int ops = k.interestOps();
            //移除CONNECT事件
            ops &= ~SelectionKey.OP_CONNECT;
            //重设事件集
            k.interestOps(ops);
			//完成连接
            unsafe.finishConnect();
        }

        // 首先处理OP_WRITE，因为我们可以写入一些队列缓冲区，从而释放内存。
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            // 调用forceFlush，它也会清除OP_WRITE，一旦没有任何东西可以写了
            ch.unsafe().forceFlush();
        }

        // 还要检查readOps为0以解决可能导致旋转循环的JDK bug
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise());
    }
}
```

首先是检查一下当前SelectionKey是否有效，如果key不再有效，则需要关闭通道。如果通道已经关闭，那么调用ch.eventLoop()将会抛出异常。最后退出。

如果key有效，那么就需要进行read(…)或write(…)操作。不过在进行read(…)或write(…)操作之前，需要先调用unsafe的finishConnect()方法，用于完成连接，否则将导致抛出NotYetConnectedException异常。

首先处理OP_WRITE事件，目的是为了可以将一些数据写入队列缓冲区，从而释放内存。

然后处理OP_READ和OP_ACCEPT事件，调用unsafe.read()进行处理。注意，此时Unsafe的实现是多态的。对于NioServerSocketChannel是交由AbstractNioMessageChannel#NioMessageUnsafe#read()处理。对于NioSocketChannel是交由AbstractNioByteChannel#NioByteUnsafe#read()处理。

另外，我们也可以想到， 对于OP_READ和OP_ACCEPT事件，它们的处理方式肯定不同。实际上，调用的都是Unsafe#read()方法，只是对于OP_READ和OP_ACCEPT事件是交由不同的ChannelInboundHandler处理。例如，对于OP_ACCEPT事件，实际上是交由位于ServerBootstrap中的内部类ServerBootstrapAcceptor处理的。

<font>channel由NioEventLoop#register()注册</font>

```java
private static void processSelectedKey(SelectionKey k, NioTask<SelectableChannel> task) {
    int state = 0;
    try {
        task.channelReady(k.channel(), k);
        state = 1;
    } catch (Exception e) {
        k.cancel();
        invokeChannelUnregistered(task, k, e);
        state = 2;
    } finally {
        switch (state) {
            case 0:
                k.cancel();
                invokeChannelUnregistered(task, k, null);
                break;
            case 1:
                if (!k.isValid()) { // Cancelled by channelReady()
                    invokeChannelUnregistered(task, k, null);
                }
                break;
            default:
                break;
        }
    }
}

private static void invokeChannelUnregistered(NioTask<SelectableChannel> task, SelectionKey k, Throwable cause) {
    try {
        task.channelUnregistered(k.channel(), cause);
    } catch (Exception e) {
        logger.warn("Unexpected exception while running NioTask.channelUnregistered()", e);
    }
}
```

对于由于NioEventLoop#register()注册Channel的事件的处理很简单，就是当有I/O事件发生时，回调NioTask#channelReady()方法，如果执行失败或完毕，则回调channelReady#channelUnregistered()方法。至于具体如何处理，则由用户基础NioTask接口自定义实现。

#### 注册channel

NioEventLoop提供的channel注册的功能。需要注意的是，虽然事件循环的Selector就位于NioEventLoop中，但是通道的的注册并不是通过NioEventLoop中的register方法注册，而是通过NioServerSocketChannel中的Unsafe内部类中的register方法注册。

这个方法的作用是注册一个任意的SelectableChannel（这个SelectableChannel不一定是由Netty创建）到这个事件循环中的Selector。当这个注册好的SelectableChannel有事件发生时，指定任务将由该事件循环执行。

NioEventLoop中的register()并没有任何接口的实现，因此，要使用该方法，需要得到NioEventLoop实例。

NioEventLoop#register()包含三个参数，分别是：

1. SelectableChannel ch：被注册的通道。
2. int interestOps：关注的事件。
3. NioTask<?> task：附加对象，即通道产生事件时，会回调NioTask方法。在NioTask中实现对I/O事件的处理。

NioEventLoop#register()源码如下：

```java
public void register(final SelectableChannel ch, final int interestOps, final NioTask<?> task) {
    ObjectUtil.checkNotNull(ch, "ch");
    if (interestOps == 0) {
        throw new IllegalArgumentException("interestOps must be non-zero.");
    }
    // 如果interestOps事件已经注册，则不能再注册了
    if ((interestOps & ~ch.validOps()) != 0) {
        throw new IllegalArgumentException(
            "invalid interestOps: " + interestOps + "(validOps: " + ch.validOps() + ')');
    }
    ObjectUtil.checkNotNull(task, "task");

    if (isShutdown()) {
        throw new IllegalStateException("event loop shut down");
    }

    if (inEventLoop()) { // 当前Executor线程，直接同步注册
        register0(ch, interestOps, task);
    } else {
        // 非当前Executor线程，交由EventLoop以任务的方式注册。
        try {
            // 卸载到EventLoop，否则java.nio.channels.spi.AbstractSelectableChannel.register
            // 在试图获取在select时可能被持有的内部锁时可能会阻塞很长时间。
            submit(new Runnable() {
                @Override
                public void run() {
                    register0(ch, interestOps, task);
                }
            }).sync();
        } catch (InterruptedException ignore) {
            // 即使被中断，我们也会对它进行调度，所以只需将线程标记为已中断。
            Thread.currentThread().interrupt();
        }
    }
}

private void register0(SelectableChannel ch, int interestOps, NioTask<?> task) {
    try {
        ch.register(unwrappedSelector, interestOps, task);
    } catch (Exception e) {
        throw new EventLoopException("failed to register a channel", e);
    }
}
```










## 创建ServerSocketChannel

## 注册Selector

## 监听Selector事件

## 处理SelectionKey事件

### Acceptor

### Read

### Write

## 运行所有任务



## ChannelPipeline



## ChannelHandler







## 附录



### Netty系统变量

- `sun.nio.ch.bugLevel`：设置NIO的BUG级别
- `io.netty.selectorAutoRebuildThreshold`：设置Selector重建阈值。