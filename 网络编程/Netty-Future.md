# Future

Future最早来源于JDK的java.util.concurrent.Future，它的作用是代表异步操作的结果。Netty定义了自己的Future，由于Netty的所有IO操作都是异步的，因此Netty Future的作用是代表Netty异步IO操作的结果。

## JDK Future

JDK Future位于java.util.concurrent.Future。源码如下：

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

java.util.concurrent.Future的DOC对其描述如下：

Future表示异步计算的结果。方法用于检查计算是否完成、等待计算完成以及检索计算结果。只有在计算完成后，才能使用get方法检索结果，在必要时阻塞直到它准备好。取消是由cancel方法执行的。还提供了其他方法来确定任务是正常完成还是被取消。一旦计算完成，就不能取消计算。如果为了可取消性而使用Future，但不提供可用的结果，可以声明Future<?> 并返回null作为底层任务的结果。

```java
interface ArchiveSearcher {
    String search(String target);
}

class App {
    ExecutorService executor = ...
    ArchiveSearcher searcher = ...
        
    void showSearch(final String target) throws InterruptedException {
        Future<String> future = executor.submit(new Callable<String>() {
            public String call() {
                return searcher.search(target);
            }
        });
        displayOtherThings(); // 搜索时做其他事情
        try {
            displayText(future.get()); // 使用future
        } catch (ExecutionException ex) {
            cleanup();
            return;
        }
    }
}
```

`FutureTask`类是实现了Runnable的Future的实现，因此可以由Executor执行。例如，上述带有submit的构造可以替换为:

```java
FutureTask<String> future =
   new FutureTask<String>(new Callable<String>() {
     public String call() {
       return searcher.search(target);
   }});
 executor.execute(future);
```

内存一致性影响：异步计算所采取的操作发生在另一个线程中相应Future.get()之后的操作之前。

<font>java.util.concurrent.Future方法：</font>

- `boolean cancel(boolean mayInterruptIfRunning)`：试图取消此任务的执行。如果任务已经完成、已经取消或由于其他原因无法取消，则此尝试将失败。如果成功，并且在调用cancel时这个任务还没有启动，那么这个任务就不应该运行。如果任务已经开始，那么`mayInterruptIfRunning`参数决定是否应该中断正在执行此任务的线程以试图停止该任务。

  该方法返回后，对`isDone`的后续调用将始终返回true。如果该方法返回true，则对`isCancelled`的后续调用将始终返回true。

- `boolean isCancelled()`：如果该任务在正常完成之前被取消，则返回true。

- `boolean isDone()`：如果任务完成，返回true。完成可能是由于正常终止、异常或取消——在所有这些情况下，此方法将返回true。

- `V get()`：阻塞等待计算完成，然后获取其结果。

- `V get(long timeout, TimeUnit unit)`：阻塞等待给定的时间以完成计算，然后获取其结果（如果有）。

## Netty Future

Netty Future位于包`io.netty.util.concurrent.Future`，其继承了`java.util.concurrent.Future`。相比于JDK Future，Netty Future还提供了监听器，等待，同步等操作。源码如下：

```java
@SuppressWarnings("ClassNameSameAsAncestorName")
public interface Future<V> extends java.util.concurrent.Future<V> {
    //当且仅当I/O操作成功完成时返回true。
    boolean isSuccess();
    //当且仅当操作可以通过cancel(boolean)取消时返回true。
    boolean isCancellable();
    //如果I/O操作失败，返回I/O操作失败的原因。
    Throwable cause();
    Adds the specified listener to this future. The specified listener is notified when this future is done. If this future is already completed, the specified listener is notified immediately.
    //将指定的监听器添加到此future。当future完成时，将通知指定的监听器。如果这个future已经完成，将立即通知指定的监听器。
    Future<V> addListener(GenericFutureListener<? extends Future<? super V>> listener);
    Future<V> addListeners(GenericFutureListener<? extends Future<? super V>>... listeners);
    //从此以后删除指定侦听器的第一个出现。当future完成时，指定的侦听器不再被通知。如果指定的侦听器与此future不关联，则此方法不执行任何操作并以静默方式返回。
    Future<V> removeListener(GenericFutureListener<? extends Future<? super V>> listener);
    //从此以后删除每个侦听器的第一次出现。当future完成时，指定的监听器不再被通知。如果指定的侦听器与此future不关联，则此方法不执行任何操作并以静默方式返回。
    Future<V> removeListeners(GenericFutureListener<? extends Future<? super V>>... listeners);
    //等待该future直到完成，如果该future失败，则重新抛出失败的原因。
    Future<V> sync() throws InterruptedException;
    Future<V> syncUninterruptibly();
    //等待着这个future的完成。
    Future<V> await() throws InterruptedException;
    //等待着这个future的完成，没有中断。此方法捕获InterruptedException并将其静默丢弃。
    Future<V> awaitUninterruptibly();
    //等待这个future在指定的时间内完成。
    boolean await(long timeout, TimeUnit unit) throws InterruptedException;
    boolean await(long timeoutMillis) throws InterruptedException;
    //等待这个future在指定的时间内完成，没有中断。此方法捕获InterruptedException并将其静默丢弃。
    boolean awaitUninterruptibly(long timeout, TimeUnit unit);
    boolean awaitUninterruptibly(long timeoutMillis);
    //不阻塞地返回结果。如果future尚未完成，则返回null。由于可能使用空值来标记future成功，您还需要检查future是否真的使用isDone()完成，而不依赖于返回的空值。
    V getNow();
    @Override
    boolean cancel(boolean mayInterruptIfRunning);
}
```

## ChannelFuture

ChannelFuture的作用是获取异步Channel I/O操作的结果。它有四种状态，分别是未完成(Uncompleted)，成功完成(Completed successfully)，失败完成(Completed with failure)，取消完成(Completed by cancellation)。

<font>其DOC描述如下：</font>

Netty中的所有I/O操作都是异步的。这意味着任何I/O调用将立即返回，而不保证所请求的I/O操作在调用结束时已经完成。相反，将返回一个`ChannelFuture`实例，它为您提供有关I/O操作的结果或状态的信息。

`ChannelFuture`不是未完成(uncompleted)就是已完成(completed)。当I/O操作开始时，将创建一个新的future对象。新的future最初是未完成(uncompleted)的—它既没有成功(succeeded)，也没有失败(failed)，也没有取消(cancelled)，因为I/O操作还没有完成。如果I/O操作成功地完成了，或者失败了，或者取消了，那么future操作将被标记为完成(completed)，并提供更具体的信息，比如失败的原因。请注意，即使失败和取消也属于完成(completed)状态。

```
                                        +---------------------------+
                                        | Completed successfully    |
                                        +---------------------------+
                                   +---->      isDone() = true      |
   +--------------------------+    |    |   isSuccess() = true      |
   |        Uncompleted       |    |    +===========================+
   +--------------------------+    |    | Completed with failure    |
   |      isDone() = false    |    |    +---------------------------+
   |   isSuccess() = false    |----+---->      isDone() = true      |
   | isCancelled() = false    |    |    |       cause() = non-null  |
   |       cause() = null     |    |    +===========================+
   +--------------------------+    |    | Completed by cancellation |
                                   |    +---------------------------+
                                   +---->      isDone() = true      |
                                        | isCancelled() = true      |
                                        +---------------------------+
```

`ChannelFuture`提供了各种方法来检查I/O操作是否已经完成(completed)、等待完成并检索I/O操作的结果。它还允许您添加`ChannelFutureListeners`，以便在I/O操作完成时获得通知。

<font>将addListener(GenericFutureListener)设置为await()</font>

建议将`addListener(GenericFutureListener)`优先用于`await()`，以便在I/O操作完成时获得通知，并执行任何后续任务。

`addListener(GenericFutureListener)`非阻塞。它只是将指定的`ChannelFutureListener`添加到`ChannelFuture`，当与future相关的I/O操作完成时，I/O线程将通知侦听器。`ChannelFutureListener`产生了最好的性能和资源利用率，因为它根本不阻塞，但如果您不习惯事件驱动编程，实现顺序逻辑可能会很棘手。

相比之下，`await()`是一个阻塞操作。一旦被调用，调用方线程将阻塞，直到操作完成。使用`await()`实现顺序逻辑更容易，但是调用方线程在I/O操作完成之前会不必要地阻塞，并且线程间通知的开销相对较高。此外，在特定的情况下存在死锁的可能性，如下所述。

<font>不要再ChannelHandler内调用await()</font>

`ChannelHandler`中的事件处理程序方法通常由I/O线程调用。如果`await()`是由I/O线程调用的事件处理程序方法调用的，那么它所等待的I/O操作可能永远不会完成，因为`await()`会阻塞它所等待的I/O操作，这就是死锁。

```java
// Bad - 永远不要这样做
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ChannelFuture future = ctx.channel().close();
    future.awaitUninterruptibly();
    // 执行post-closure操作
    // ...
}

// Good
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ChannelFuture future = ctx.channel().close();
    future.addListener(new ChannelFutureListener() {
        public void operationComplete(ChannelFuture future) {
            // 执行post-closure操作
            // ...
        }
    });
}
```

尽管有上面提到的缺点，但确实存在调用`await()`更方便的情况。在这种情况下，请确保您没有在I/O线程中调用`await()`。否则，将引发`BlockingOperationException`以防止死锁。

<font>不要混淆I/O超时和等待超时</font>

使用`await(long)`、`await(long, TimeUnit)`、`awaituninterruptible(long)`或`awaituninterruptible(long, TimeUnit)`指定的超时值与I/O超时无关。如果I/O操作超时，future将被标记为“失败完成（completed with failure）”，如上图所示。例如，连接超时应该通过传输特定的选项配置：

```java
// BAD - NEVER DO THIS
Bootstrap b = ...;
ChannelFuture f = b.connect(...);
f.awaitUninterruptibly(10, TimeUnit.SECONDS);
if (f.isCancelled()) {
    // Connection attempt cancelled by user
} else if (!f.isSuccess()) {
    // You might get a NullPointerException here because the future
    // might not be completed yet.
    f.cause().printStackTrace();
} else {
    // Connection established successfully
}

// GOOD
Bootstrap b = ...;
// Configure the connect timeout option.
b.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 10000);
ChannelFuture f = b.connect(...);
f.awaitUninterruptibly();

// Now we are sure the future is completed.
assert f.isDone();

if (f.isCancelled()) {
    // Connection attempt cancelled by user
} else if (!f.isSuccess()) {
    f.cause().printStackTrace();
} else {
    // Connection established successfully
}
```

ChannelFuture源码：

```java
public interface ChannelFuture extends Future<Void> {
    Channel channel();
    @Override
    ChannelFuture addListener(GenericFutureListener<? extends Future<? super Void>> listener);
    @Override
    ChannelFuture addListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
    @Override
    ChannelFuture removeListener(GenericFutureListener<? extends Future<? super Void>> listener);
    @Override
    ChannelFuture removeListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
    @Override
    ChannelFuture sync() throws InterruptedException;
    @Override
    ChannelFuture syncUninterruptibly();
    @Override
    ChannelFuture await() throws InterruptedException;
    @Override
    ChannelFuture awaitUninterruptibly();
    boolean isVoid();
}
```

可以看到ChannelFuture继承了`io.netty.util.concurrent.Future`，并重写了其中大部分方法，目的是为了通过ChannelFuture接口提供相似的行为。除此之外，ChannelFuture声明了两个抽象方法：

- `Channel channel()`：

  返回与此future关联的I/O操作发生的通道。

- `boolean isVoid()`：

  如果这个ChannelFuture是一个void future，因此不允许调用以下任何方法，则返回true:

  - addListener(GenericFutureListener)
  - addListeners(GenericFutureListener[])
  - await()
  - await(long, TimeUnit) ()}
  - await(long) ()}
  - awaitUninterruptibly()
  - sync()
  - syncUninterruptibly()

## AbstractFuture

java.util.concurrent.Future提供了两个get方法用于同步阻塞获取异步操作的结果。在Netty中，其实现位于io.netty.util.concurrent.AbstractFuture：

```java
public abstract class AbstractFuture<V> implements Future<V> {

    @Override
    public V get() throws InterruptedException, ExecutionException {
        await();

        Throwable cause = cause();
        if (cause == null) {
            return getNow();
        }
        if (cause instanceof CancellationException) {
            throw (CancellationException) cause;
        }
        throw new ExecutionException(cause);
    }

    @Override
    public V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException {
        if (await(timeout, unit)) {
            Throwable cause = cause();
            if (cause == null) {
                return getNow();
            }
            if (cause instanceof CancellationException) {
                throw (CancellationException) cause;
            }
            throw new ExecutionException(cause);
        }
        throw new TimeoutException();
    }
}
```

两个get方法的实现大同小异，首先调用await方法进行有限或无限阻塞，当I/O操作完成之后会被notify()唤醒，程序继续向下执行，检查I/O操作是否发生异常，如果没有发生异常，则调用getNow()获取结果。否则，将异常堆栈进行包装，抛出相应异常。对于await有限阻塞，如果超时的话，将抛出TimeoutException异常。

## Netty Promise

Netty Promise接口位于包io.netty.util.concurrent.Promise，继承`io.netty.util.concurrent.Future`。与Future接口相比，Future接口并没有提供写操作的相关功能，Promise接口补足了这一块儿。想象一下，在每次I/O操作调用结束返回时，必然需要一个接口去设置调用的结果和状态，Promise接口就是负责这件事的。

Promise源码如下：

```java
public interface Promise<V> extends Future<V> {
    Promise<V> setSuccess(V result);
    boolean trySuccess(V result);
    Promise<V> setFailure(Throwable cause);
    boolean tryFailure(Throwable cause);
    boolean setUncancellable();
    @Override
    Promise<V> addListener(GenericFutureListener<? extends Future<? super V>> listener);
    @Override
    Promise<V> addListeners(GenericFutureListener<? extends Future<? super V>>... listeners);
    @Override
    Promise<V> removeListener(GenericFutureListener<? extends Future<? super V>> listener);
    @Override
    Promise<V> removeListeners(GenericFutureListener<? extends Future<? super V>>... listeners);
    @Override
    Promise<V> await() throws InterruptedException;
    @Override
    Promise<V> awaitUninterruptibly();
    @Override
    Promise<V> sync() throws InterruptedException;
    @Override
    Promise<V> syncUninterruptibly();
}
```

与`io.netty.util.concurrent.Future`相比，Promise接口另外提供了如下抽象方法：

- `Promise<V> setSuccess(V result)`：标记着这个future的成功，并通知所有的侦听器。如果已经成功或失败，它将抛出一个IllegalStateException。
- `boolean trySuccess(V result)`：标记着这个future的成功，并通知所有的侦听器。
- `Promise<V> setFailure(Throwable cause)`：将此future标记为失败并通知所有侦听器。如果已经成功或失败，它将抛出一个IllegalStateException。
- `boolean tryFailure(Throwable cause)`：将此future标记为失败并通知所有侦听器。
- `boolean setUncancellable()`：让这个future无法取消。

## ChannelPromise

ChannelPromise源码如下：

```java
public interface ChannelPromise extends ChannelFuture, Promise<Void> {
    @Override
    Channel channel();
    @Override
    ChannelPromise setSuccess(Void result);
    ChannelPromise setSuccess();
    boolean trySuccess();
    @Override
    ChannelPromise setFailure(Throwable cause);
    @Override
    ChannelPromise addListener(GenericFutureListener<? extends Future<? super Void>> listener);
    @Override
    ChannelPromise addListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
    @Override
    ChannelPromise removeListener(GenericFutureListener<? extends Future<? super Void>> listener);
    @Override
    ChannelPromise removeListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
    @Override
    ChannelPromise sync() throws InterruptedException;
    @Override
    ChannelPromise syncUninterruptibly();
    @Override
    ChannelPromise await() throws InterruptedException;
    @Override
    ChannelPromise awaitUninterruptibly();
    /**
     * Returns a new {@link ChannelPromise} if {@link #isVoid()} returns {@code true} otherwise itself.
     */
    ;
}
```

ChannelPromise与Promise没有太多差别，只是额外提供了两个方法：

1. ChannelPromise setSuccess()：标记着这个future的成功，并通知所有的侦听器。至于设置的值，默认以NULL进行设置。
2. ChannelPromise unvoid()：如果isVoid()返回true，即是一个void future。则返回一个新的ChannelPromise，否则返回自身。

## DefaultChannelPromise

DefaultChannelPromise是一个比较标准的实现，它的类型层次如下图所示：

![]()

实际上，DefaultChannelPromise并没有任何有效的内容，大部分都是调用基类DefaultPromise。在DefaultChannelPromise中，主要包持有了对Channel和checkpoint的引用。虽然Future主要的作用是获取异步I/O操作结果，不过在其中持久一些操作需要对象的引用，以方便获取，也是没有问题的。

DefaultChannelPromise源码如下：

```java
public class DefaultChannelPromise extends DefaultPromise<Void> implements ChannelPromise, FlushCheckpoint {

    private final Channel channel;
    private long checkpoint;
	......
}
```

### DefaultPromise

```java
public class DefaultPromise<V> extends AbstractFuture<V> implements Promise<V> {
    private static final InternalLogger logger = InternalLoggerFactory.getInstance(DefaultPromise.class);
    private static final InternalLogger rejectedExecutionLogger =
            InternalLoggerFactory.getInstance(DefaultPromise.class.getName() + ".rejectedExecution");
    //最大侦听器栈深度，默认为8
    private static final int MAX_LISTENER_STACK_DEPTH = Math.min(8,
           SystemPropertyUtil.getInt("io.netty.defaultPromise.maxListenerStackDepth", 8));
    // 更新result的CAS更新器
    @SuppressWarnings("rawtypes")
    private static final AtomicReferenceFieldUpdater<DefaultPromise, Object> RESULT_UPDATER =
            AtomicReferenceFieldUpdater.newUpdater(DefaultPromise.class, Object.class, "result");
    // 成功时，如果设置result为NULL，则会将结果设置为当前SUCCESS
    private static final Object SUCCESS = new Object();
    // 使Future不可取消时，设置result为UNCANCELLABLE
    private static final Object UNCANCELLABLE = new Object();
    // 取消时，result值为CANCELLATION_CAUSE_HOLDER
    private static final CauseHolder CANCELLATION_CAUSE_HOLDER = new CauseHolder(
            StacklessCancellationException.newInstance(DefaultPromise.class, "cancel(...)"));
    private static final StackTraceElement[] CANCELLATION_STACK = CANCELLATION_CAUSE_HOLDER.cause.getStackTrace();
	// I/O调用结果
    private volatile Object result;
    // EventLoop
    private final EventExecutor executor;
    /**
     * 一个或多个侦听器。可以是GenericFutureListener或DefaultFutureListeners。
     * 如果为空，则意味着1)没有添加侦听器，或者2)通知了所有侦听器。
     * Threading - synchronized(this).当没有EventExecutor时，我们必须支持添加侦听器。
     */
    private Object listeners;
    /**
     * Threading - synchronized(this). 
     * 我们需要持有监视器，以使用Java的底层wait()/notifyAll()。
     * 统计当前阻塞的线程数。
     */
    private short waiters;
    /**
     * Threading - synchronized(this). 
     * 如果执行器发生变化，我们必须防止并发通知和FIFO侦听器通知。
     */
    private boolean notifyingListeners;
	......
}
```

#### 添加侦听器

addListener实现如下：

```java
@Override
public Promise<V> addListener(GenericFutureListener<? extends Future<? super V>> listener) {
    checkNotNull(listener, "listener");
    synchronized (this) {
        addListener0(listener);
    }
    if (isDone()) {
        notifyListeners();
    }

    return this;
}

@Override
public Promise<V> addListeners(GenericFutureListener<? extends Future<? super V>>... listeners) {
    checkNotNull(listeners, "listeners");
    synchronized (this) {
        for (GenericFutureListener<? extends Future<? super V>> listener : listeners) {
            if (listener == null) {
                break;
            }
            addListener0(listener);
        }
    }
    if (isDone()) {
        notifyListeners();
    }
    return this;
}
```

两个实现都差不多，只是一个是同一时间只能添加一个侦听器，而另一个可以同时添加多个。最后都是一个一个调用addListener0()添加。添加完成之后判断I/O操作是否已经完成，如果没有，则直接返回当前Future。如果已经完成，那么就需要马上回调侦听器，因此，调用notifyListeners()唤醒所有阻塞的线程并回调侦听器。

<font>addListener0</font>

```java
private void addListener0(GenericFutureListener<? extends Future<? super V>> listener) {
    if (listeners == null) {
        listeners = listener;
    } else if (listeners instanceof DefaultFutureListeners) {
        ((DefaultFutureListeners) listeners).add(listener);
    } else {
        listeners = new DefaultFutureListeners((GenericFutureListener<?>) listeners, listener);
    }
}
```

如果是第一个侦听器，则直接赋值给listeners，如果是第二个侦听器，则将第一个和第二个一起包装为一个DefaultFutureListeners对象，然后赋值给listeners。如果是第三个即以上，则调用DefaultFutureListeners的add方法将其添加进去。

<font>notifyListeners</font>

```java
private void notifyListeners() {
    EventExecutor executor = executor(); // 事件循环EventLoop
    if (executor.inEventLoop()) {
        final InternalThreadLocalMap threadLocals = InternalThreadLocalMap.get();
        final int stackDepth = threadLocals.futureListenerStackDepth();
        // 栈深度小于MAX_LISTENER_STACK_DEPTH（默认为8）
        if (stackDepth < MAX_LISTENER_STACK_DEPTH) {
            // 栈深度 + 1
            threadLocals.setFutureListenerStackDepth(stackDepth + 1);
            try {
                //唤醒回调
                notifyListenersNow();
            } finally {
                threadLocals.setFutureListenerStackDepth(stackDepth);
            }
            return;
        }
    }
	// 到此处，要么是唤醒线程不是EventExecutor线程，要么是栈深度大于等于MAX_LISTENER_STACK_DEPTH，交由事件循环以普通任务的形式在运行时执行。
    safeExecute(executor, new Runnable() {
        @Override
        public void run() {
            notifyListenersNow();
        }
    });
}
```

唤醒侦听器的操作，如果当前线程不是EventExecutor线程，那么将交由事件循环以普通任务的形式在运行时执行。另外，如果当前线程是EventExecutor线程，在当前线程中不超过8次唤醒侦听器的操作同步执行，否则，超过8次的部分，仍然是交由事件循环以普通任务的形式在运行时执行。

<font>notifyListenersNow</font>

```java
private void notifyListenersNow() {
    Object listeners;
    synchronized (this) {
        // 只有在有侦听器要通知而我们还没有通知侦听器的情况下才继续。
        if (notifyingListeners || this.listeners == null) {
            return;
        }
        notifyingListeners = true;
        listeners = this.listeners;
        this.listeners = null;
    }
    for (;;) {
        if (listeners instanceof DefaultFutureListeners) {
            notifyListeners0((DefaultFutureListeners) listeners);
        } else {
            notifyListener0(this, (GenericFutureListener<?>) listeners);
        }
        synchronized (this) {
            if (this.listeners == null) {
                // 在这个方法中不能抛出任何东西，
                // 所以不需要在finally块中将notifyingListeners设置为false。
                notifyingListeners = false;
                return;
            }
            listeners = this.listeners;
            this.listeners = null;
        }
    }
}
```

如果没有侦听器，则直接返回。接着，根据listeners类型选择不同的分支（如果类型是DefaultFutureListeners，则表示有多个侦听器，如果不是，则表示只有一个侦听器），然后调用重载的notifyListener0方法，其本质都是一样的。最后，由于侦听器的加添可能是并发的，因此还需要再判断一下在回调侦听器的过程中是否被添加了新的侦听器，如果是，则继续重复前面的流程。

<font>notifyListeners0</font>

```java
private void notifyListeners0(DefaultFutureListeners listeners) {
    GenericFutureListener<?>[] a = listeners.listeners();
    int size = listeners.size();
    for (int i = 0; i < size; i ++) {
        notifyListener0(this, a[i]);
    }
}

@SuppressWarnings({ "unchecked", "rawtypes" })
private static void notifyListener0(Future future, GenericFutureListener l) {
    try {
        l.operationComplete(future);
    } catch (Throwable t) {
        if (logger.isWarnEnabled()) {
            logger.warn("An exception was thrown by " + l.getClass().getName() + ".operationComplete()", t);
        }
    }
}
```

notifyListener0是两个重载的方法，作用是回调侦听器。对于单个侦听器，直接回调，对于多个侦听器，轮询回调。

#### 删除侦听器

```java
@Override
public Promise<V> removeListener(final GenericFutureListener<? extends Future<? super V>> listener) {
    checkNotNull(listener, "listener");
    synchronized (this) {
        removeListener0(listener);
    }
    return this;
}

@Override
public Promise<V> removeListeners(final GenericFutureListener<? extends Future<? super V>>... listeners) {
    checkNotNull(listeners, "listeners");
    synchronized (this) {
        for (GenericFutureListener<? extends Future<? super V>> listener : listeners) {
            if (listener == null) {
                break;
            }
            removeListener0(listener);
        }
    }
    return this;
}
```

删除侦听器的操作很简单，如果是单个侦听器，直接调用removeListener0删除，如果是多个侦听器，轮询调用removeListener0删除。

removeListener0

```java
private void removeListener0(GenericFutureListener<? extends Future<? super V>> listener) {
    if (listeners instanceof DefaultFutureListeners) {
        ((DefaultFutureListeners) listeners).remove(listener);
    } else if (listeners == listener) {
        listeners = null;
    }
}
```

单个侦听器，直接将listeners设置为NULL；多个的话，调用DefaultFutureListeners的remove方法删除。

#### 设置值

setSuccess和setFailure

```java
@Override
public Promise<V> setSuccess(V result) {
    if (setSuccess0(result)) {
        return this;
    }
    throw new IllegalStateException("complete already: " + this);
}

@Override
public boolean trySuccess(V result) {
    return setSuccess0(result);
}

@Override
public Promise<V> setFailure(Throwable cause) {
    if (setFailure0(cause)) {
        return this;
    }
    throw new IllegalStateException("complete already: " + this, cause);
}

@Override
public boolean tryFailure(Throwable cause) {
    return setFailure0(cause);
}
```

setSuccess和setFailure很简单，差异在于，对于try而言，返回一个布尔值，表示成功或失败。而对于非try而言，设置成功返回当前Future对象，设置失败抛出IllegalStateException异常。

setSuccess0

```java
private boolean setSuccess0(V result) {
    return setValue0(result == null ? SUCCESS : result);
}

private boolean setFailure0(Throwable cause) {
    return setValue0(new CauseHolder(checkNotNull(cause, "cause")));
}
```

setValue0

```java
private boolean setValue0(Object objResult) {
    if (RESULT_UPDATER.compareAndSet(this, null, objResult) ||
        RESULT_UPDATER.compareAndSet(this, UNCANCELLABLE, objResult)) {
        if (checkNotifyWaiters()) {
            notifyListeners();
        }
        return true;
    }
    return false;
}

private synchronized boolean checkNotifyWaiters() {
    if (waiters > 0) {
        notifyAll();
    }
    return listeners != null;
}
```

当Future未完成(result=null)，或未取消(result=UNCANCELLABLE)，则将result设置为设置值objResult，然后唤醒所有阻塞的线程(get)，并通知所有侦听器。

#### 获取值

DefaultPromise重写了在AbstractFuture中实现的get()方法：

```java
@SuppressWarnings("unchecked")
@Override
public V get() throws InterruptedException, ExecutionException {
    Object result = this.result;
    if (!isDone0(result)) {
        await();
        result = this.result;
    }
    if (result == SUCCESS || result == UNCANCELLABLE) {
        return null;
    }
    Throwable cause = cause0(result);
    if (cause == null) {
        return (V) result;
    }
    if (cause instanceof CancellationException) {
        throw (CancellationException) cause;
    }
    throw new ExecutionException(cause);
}

@SuppressWarnings("unchecked")
@Override
public V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException {
    Object result = this.result;
    if (!isDone0(result)) {
        if (!await(timeout, unit)) {
            throw new TimeoutException();
        }
        result = this.result;
    }
    if (result == SUCCESS || result == UNCANCELLABLE) {
        return null;
    }
    Throwable cause = cause0(result);
    if (cause == null) {
        return (V) result;
    }
    if (cause instanceof CancellationException) {
        throw (CancellationException) cause;
    }
    throw new ExecutionException(cause);
}
```

一个无限阻塞获取，一个有限阻塞获取。流程如下：

1. 判断I/O操作是否已经完成，如果未完成，则调用await挂起线程。
2. 如果已经完成，并且result等于SUCCESS或UNCANCELLABLE，则直接返回NULL。
3. 如果不存在异常，则返回结果。
4. 如果存在异常，则抛出异常。

#### 阻塞等待

阻塞等待分两类，一类是无限阻塞，一类是有限时间阻塞。

<font>无限阻塞</font>

```java
@Override
public Promise<V> await() throws InterruptedException {
    if (isDone()) {
        return this;
    }
    if (Thread.interrupted()) {
        throw new InterruptedException(toString());
    }
    checkDeadLock();
    synchronized (this) {
        while (!isDone()) {
            incWaiters();
            try {
                wait();
            } finally {
                decWaiters();
            }
        }
    }
    return this;
}

@Override
public Promise<V> awaitUninterruptibly() {
    if (isDone()) {
        return this;
    }
    checkDeadLock();
    boolean interrupted = false;
    synchronized (this) {
        while (!isDone()) {
            incWaiters();
            try {
                wait();
            } catch (InterruptedException e) {
                // Interrupted while waiting.
                interrupted = true;
            } finally {
                decWaiters();
            }
        }
    }
    if (interrupted) {
        Thread.currentThread().interrupt();
    }
    return this;
}

private void incWaiters() {
    if (waiters == Short.MAX_VALUE) {
        throw new IllegalStateException("too many waiters: " + this);
    }
    ++waiters;
}

private void decWaiters() {
    --waiters;
}
```

这两个无限阻塞的await()的差异在于一个会抛出中断异常，一个不会，而是法捕获InterruptedException并将其静默丢弃。至于其他实现都是一样的。

首先调用isDone()方法判断I/O操作是否已经完成，如果已经完成，那么就直接返回当前Future。为了避免并发，需要synchronized，再次判断isDone()方法，类似于单例的实现。incWaiters()和decWaiters()方法为了统计阻塞的线程数据，最大不能超过32767。然后调用Object的wait()挂起线程。

至于checkDeadLock()方法，检查了调用await()方法的线程是否是当前EventExecutor线程，要求必须不是，否则将抛出异常BlockingOperationException异常。

```java
protected void checkDeadLock() {
    EventExecutor e = executor();
    if (e != null && e.inEventLoop()) {
        throw new BlockingOperationException(toString());
    }
}
```

<font>有限时间阻塞</font>

有限时间阻塞的实现如下所示，跟无限阻塞差不多，唯一不同的是有阻塞时间限制。

```java
@Override
public boolean await(long timeout, TimeUnit unit) throws InterruptedException {
    return await0(unit.toNanos(timeout), true);
}

@Override
public boolean await(long timeoutMillis) throws InterruptedException {
    return await0(MILLISECONDS.toNanos(timeoutMillis), true);
}

@Override
public boolean awaitUninterruptibly(long timeout, TimeUnit unit) {
    try {
        return await0(unit.toNanos(timeout), false);
    } catch (InterruptedException e) {
        // Should not be raised at all.
        throw new InternalError();
    }
}

@Override
public boolean awaitUninterruptibly(long timeoutMillis) {
    try {
        return await0(MILLISECONDS.toNanos(timeoutMillis), false);
    } catch (InterruptedException e) {
        // Should not be raised at all.
        throw new InternalError();
    }
}

private boolean await0(long timeoutNanos, boolean interruptable) throws InterruptedException {
    if (isDone()) {
        return true;
    }
    if (timeoutNanos <= 0) {
        return isDone();
    }
    if (interruptable && Thread.interrupted()) {
        throw new InterruptedException(toString());
    }
    checkDeadLock();
    // 从这里开始计算时间而不是这个方法的第一行，
    // 以避免/推迟System.nanoTime()的性能成本。
    final long startTime = System.nanoTime();
    synchronized (this) {
        boolean interrupted = false;
        try {
            long waitTime = timeoutNanos;
            while (!isDone() && waitTime > 0) {
                incWaiters();
                try {
                    wait(waitTime / 1000000, (int) (waitTime % 1000000));
                } catch (InterruptedException e) {
                    if (interruptable) {
                        throw e;
                    } else {
                        interrupted = true;
                    }
                } finally {
                    decWaiters();
                }
                // 提前检查isDone()，尽量避免以后计算运行时间。
                if (isDone()) {
                    return true;
                }
                // 在这里计算消耗的时间，而不是在while条件下，
                // 尽量避免在while的第一个循环中使用System.nanoTime()的性能开销。
                waitTime = timeoutNanos - (System.nanoTime() - startTime);
            }
            return isDone();
        } finally {
            if (interrupted) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

#### 同步

sync()和syncUninterruptibly()实现如下：

```java
@Override
public Promise<V> sync() throws InterruptedException {
    await();
    rethrowIfFailed();
    return this;
}

@Override
public Promise<V> syncUninterruptibly() {
    awaitUninterruptibly();
    rethrowIfFailed();
    return this;
}

private void rethrowIfFailed() {
    Throwable cause = cause();
    if (cause == null) {
        return;
    }
    PlatformDependent.throwException(cause);
}
```

同步实现没有太多可说的，主要就是调用await()或awaitUninterruptibly()挂起线程。主要是最后阻塞如果是由于异常被唤醒，将会抛出异常。

#### 取消

```java
@Override
public boolean cancel(boolean mayInterruptIfRunning) {
    if (RESULT_UPDATER.compareAndSet(this, null, CANCELLATION_CAUSE_HOLDER)) {
        if (checkNotifyWaiters()) {
            notifyListeners();
        }
        return true;
    }
    return false;
}
```

取消操作的参数mayInterruptIfRunning并没有被使用。首先是将result通过CAS设置为CANCELLATION_CAUSE_HOLDER，因此取消时，可能已完成，也可能为完成，所以需要通过CAS设置为CANCELLATION_CAUSE_HOLDER。当设置为CANCELLATION_CAUSE_HOLDER成功时意味着取消成功，因此阻塞的线程就不需要再阻塞了，有侦听器的则需要回调侦听器，所以接下来检查是否存在阻塞线程，如果存在，则唤醒所有的阻塞线程，并通知所有侦听器。

#### 异常

```java
@Override
public Throwable cause() {
    return cause0(result);
}

private Throwable cause0(Object result) {
    if (!(result instanceof CauseHolder)) {
        return null;
    }
    if (result == CANCELLATION_CAUSE_HOLDER) {
        CancellationException ce = new LeanCancellationException();
        if (RESULT_UPDATER.compareAndSet(this, CANCELLATION_CAUSE_HOLDER, new CauseHolder(ce))) {
            return ce;
        }
        result = this.result;
    }
    return ((CauseHolder) result).cause;
}
```

异常是被包装在内部类CauseHolder中的，因此，如果result不是CauseHolder，则表示没有异常，因而直接返回NULL。如果是，那么它可能是CANCELLATION_CAUSE_HOLDER，即是被取消，此时返回被取消的异常。反之，就返回对应CauseHolder包装的异常。

## VoidChannelPromise

VoidChannelPromise是一个占位符对象，因此它可以在不同的操作中重用。由于VoidChannelPromise的所有实现都是空实现，因此使用它不会得到任何通知。用于在使用ChannelPromise作为参数，但不希望得到任何通知的地方。

虽然VoidChannelPromise不能得到任何通知，但是它可以触发异常。是ChannelHandler的exceptionCaught方法收到异常信息。

VoidChannelPromise源码如下：

```java
@UnstableApi
public final class VoidChannelPromise extends AbstractFuture<Void> implements ChannelPromise {

    private final Channel channel;
    // 如果我们不应该在失败情况下通过管道传播异常，则将为空。
    private final ChannelFutureListener fireExceptionListener;

    public VoidChannelPromise(final Channel channel, boolean fireException) {
        ObjectUtil.checkNotNull(channel, "channel");
        this.channel = channel;
        if (fireException) {
            fireExceptionListener = new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture future) throws Exception {
                    Throwable cause = future.cause();
                    if (cause != null) {
                        fireException0(cause);
                    }
                }
            };
        } else {
            fireExceptionListener = null;
        }
    }

    ......

    @Override
    public VoidChannelPromise setFailure(Throwable cause) {
        fireException0(cause);
        return this;
    }
	
    ......

    @Override
    public boolean tryFailure(Throwable cause) {
        fireException0(cause);
        return false;
    }
	
    ......

    @Override
    public ChannelPromise unvoid() {
        ChannelPromise promise = new DefaultChannelPromise(channel);
        if (fireExceptionListener != null) {
            promise.addListener(fireExceptionListener);
        }
        return promise;
    }

    @Override
    public boolean isVoid() {
        return true;
    }

    private void fireException0(Throwable cause) {
        // 只有在通道打开并注册的情况下才触发异常，否则管道没有设置，因此它会碰到管道的尾部。
        // See https://github.com/netty/netty/issues/1517
        if (fireExceptionListener != null && channel.isRegistered()) {
            channel.pipeline().fireExceptionCaught(cause);
        }
    }
}
```

在Netty中，只有两个地方在使用VoidChannelPromise。

一个是AbstractChannel：

```java
public abstract class AbstractChannel extends DefaultAttributeMap implements Channel {
	......
    private final VoidChannelPromise unsafeVoidPromise = new VoidChannelPromise(this, false);
    ......
    @Override
    public final ChannelPromise voidPromise() {
        assertEventLoop();
        return unsafeVoidPromise;
    }
    ......
}
```

位于Channel#Unsafe接口的voidPromise()方法启动DOC描述如下：

```
返回一个特殊的ChannelPromise，它可以被重用并传递给Channel.Unsafe中的操作。它永远不会收到成功或错误的通知，因此它只是一个占位符，用于接受ChannelPromise作为参数但您不希望得到通知的操作。
```

```java
public interface Channel extends AttributeMap, ChannelOutboundInvoker, Comparable<Channel> {
    interface Unsafe {
        ChannelPromise voidPromise();
    }
}
```

另一个是DefaultChannelPipeline：

```java
public class DefaultChannelPipeline implements ChannelPipeline {
    ......
    private final VoidChannelPromise voidPromise;
    ......
    protected DefaultChannelPipeline(Channel channel) {
        ......
        voidPromise =  new VoidChannelPromise(channel, true);
        ......
    }
    ......
    @Override
    public final ChannelPromise voidPromise() {
        return voidPromise;
    }
    ......
}
```

其位于ChannelOutboundInvoker接口的voidPromise()方法启动DOC描述如下：

```
	返回一个特殊的ChannelPromise，它可以在不同的操作中重用。
	它只支持将其用于write(Object, ChannelPromise)。
	请注意，返回的ChannelPromise将不支持大多数操作，只有在您希望为每个写操作保存一个对象分配时才应该使用。您将无法检测操作是否完成，只能检测它是否失败，因为在这种情况下，实现将调用ChannelPipeline.fireExceptionCaught(Throwable)。
	注意，这是一个专家特性，应该小心使用!
```

```java
public interface ChannelOutboundInvoker {
    ......
    ChannelPromise voidPromise();
    ......
}
```