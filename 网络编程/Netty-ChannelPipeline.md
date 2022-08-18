# ChannelPipeline





```java
public interface ChannelInboundInvoker {
    ChannelInboundInvoker fireChannelRegistered();
    ChannelInboundInvoker fireChannelUnregistered();
    ChannelInboundInvoker fireChannelActive();
    ChannelInboundInvoker fireChannelInactive();
    ChannelInboundInvoker fireExceptionCaught(Throwable cause);
    ChannelInboundInvoker fireUserEventTriggered(Object event);
    ChannelInboundInvoker fireChannelRead(Object msg);
    ChannelInboundInvoker fireChannelReadComplete();
    ChannelInboundInvoker fireChannelWritabilityChanged();
}
```





```java
public interface ChannelOutboundInvoker {
    ChannelFuture bind(SocketAddress localAddress);
    ChannelFuture connect(SocketAddress remoteAddress);
    ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress);
    ChannelFuture disconnect();
    ChannelFuture close();
    ChannelFuture deregister();
    ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise);
    ChannelFuture connect(SocketAddress remoteAddress, ChannelPromise promise);
    ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise);
    ChannelFuture disconnect(ChannelPromise promise);
    ChannelFuture close(ChannelPromise promise);
    ChannelFuture deregister(ChannelPromise promise);
    ChannelOutboundInvoker read();
    ChannelFuture write(Object msg);
    ChannelFuture write(Object msg, ChannelPromise promise);
    ChannelOutboundInvoker flush();
    ChannelFuture writeAndFlush(Object msg, ChannelPromise promise);
    ChannelFuture writeAndFlush(Object msg);
    ChannelPromise newPromise();
    ChannelProgressivePromise newProgressivePromise();
    ChannelFuture newSucceededFuture();
    ChannelFuture newFailedFuture(Throwable cause);
    ChannelPromise voidPromise();
}
```



## ChannelPipeline

ChannelHandlers的列表，用于处理或拦截Channel的入站事件和出站操作。ChannelPipeline实现了截取过滤器(Intercepting Filter)模式的高级形式，让用户完全控制如何处理事件，以及管道中的ChannelHandlers如何相互交互。

<font>创建管道</font>

每个通道(channel)都有自己的管道(pipeline)，并且在创建新通道时自动创建。

<font>事件如何在管道中流动</font>

下图描述了ChannelHandlers通常如何在ChannelPipeline中处理I/O事件。I/O事件由ChannelInboundHandler或ChannelOutboundHandler处理，并通过调用ChannelHandlerContext中定义的事件传播方法(如ChannelHandlerContext.firechannelread(Object)和ChannelHandlerContext.write(Object))转发到最接近的处理程序。

```
                                                  I/O Request
                                                 via Channel or
                                              ChannelHandlerContext
                                                        |
    +---------------------------------------------------+---------------+
    |                           ChannelPipeline         |               |
    |                                                  \|/              |
    |    +---------------------+            +-----------+----------+    |
    |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
    |    +----------+----------+            +-----------+----------+    |
    |              /|\                                  |               |
    |               |                                  \|/              |
    |    +----------+----------+            +-----------+----------+    |
    |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
    |    +----------+----------+            +-----------+----------+    |
    |              /|\                                  .               |
    |               .                                   .               |
    | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
    |        [ method call]                       [method call]         |
    |               .                                   .               |
    |               .                                  \|/              |
    |    +----------+----------+            +-----------+----------+    |
    |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
    |    +----------+----------+            +-----------+----------+    |
    |              /|\                                  |               |
    |               |                                  \|/              |
    |    +----------+----------+            +-----------+----------+    |
    |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
    |    +----------+----------+            +-----------+----------+    |
    |              /|\                                  |               |
    +---------------+-----------------------------------+---------------+
                    |                                  \|/
    +---------------+-----------------------------------+---------------+
    |               |                                   |               |
    |       [ Socket.read() ]                    [ Socket.write() ]     |
    |                                                                   |
    |  Netty Internal I/O Threads (Transport Implementation)            |
    +-------------------------------------------------------------------+
```

入站事件由入站处理程序按照自底向上的方向处理，如图左侧所示。入站处理程序通常处理图底部的I/O线程生成的入站数据。入站数据通常通过实际的输入操作(如SocketChannel.read(ByteBuffer))从远程对等端读取。如果入站事件超出了最上层的入站处理程序，则会以静默方式丢弃它，或者在需要注意时记录它。

出站事件由出站处理程序按自顶向下的方向处理，如图右侧所示。出站处理程序通常生成或转换出站通信流，如写请求。如果出站事件超出了底部出站处理程序，则由与Channel关联的I/O线程处理。I/O线程经常执行实际的输出操作，例如SocketChannel.write(ByteBuffer)。

例如，让我们假设我们创建了以下管道:

```java
ChannelPipeline p = ...;
p.addLast("1", new InboundHandlerA());
p.addLast("2", new InboundHandlerB());
p.addLast("3", new OutboundHandlerA());
p.addLast("4", new OutboundHandlerB());
p.addLast("5", new InboundOutboundHandlerX());
```

在上面的示例中，名称以Inbound开头的类意味着它是一个入站处理程序。名称以Outbound开头的类意味着它是一个出站处理程序。

在给定的示例配置中，当事件进入时，处理程序的评估顺序是1、2、3、4、5。当事件出站时，顺序是5、4、3、2、1。基于这一原则，ChannelPipeline跳过某些处理程序的计算以缩短堆栈深度:

- 3和4没有实现ChannelInboundHandler，因此入站事件的实际计算顺序将是：1、2和5。
- 1和2没有实现ChannelOutboundHandler，因此出站事件的实际计算顺序为：5、4和3。
- 如果5同时实现了ChannelInboundHandler和ChannelOutboundHandler，入站和出站事件的计算顺序可能分别为125和543。

<font>将事件转发到下一个处理程序</font>

正如您可能在图中注意到的，处理程序必须调用ChannelHandlerContext中的事件传播方法来将事件转发给它的下一个处理程序。这些方法包括：

- 入站事件传播方法：
  - ChannelHandlerContext.fireChannelRegistered()
  - ChannelHandlerContext.fireChannelActive()
  - ChannelHandlerContext.fireChannelRead(Object)
  - ChannelHandlerContext.fireChannelReadComplete()
  - ChannelHandlerContext.fireExceptionCaught(Throwable)
  - ChannelHandlerContext.fireUserEventTriggered(Object)
  - ChannelHandlerContext.fireChannelWritabilityChanged()
  - ChannelHandlerContext.fireChannelInactive()
  - ChannelHandlerContext.fireChannelUnregistered()
- 出站事件传播方法：
  - ChannelHandlerContext.bind(SocketAddress, ChannelPromise)
  - ChannelHandlerContext.connect(SocketAddress, SocketAddress, ChannelPromise)
  - ChannelHandlerContext.write(Object, ChannelPromise)
  - ChannelHandlerContext.flush()
  - ChannelHandlerContext.read()
  - ChannelHandlerContext.disconnect(ChannelPromise)
  - ChannelHandlerContext.close(ChannelPromise)
  - ChannelHandlerContext.deregister(ChannelPromise)

下面的例子展示了事件传播通常是如何完成的：

```java
public class MyInboundHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        System.out.println("Connected!");
        ctx.fireChannelActive();
    }
}

public class MyOutboundHandler extends ChannelOutboundHandlerAdapter {
    @Override
    public void close(ChannelHandlerContext ctx, ChannelPromise promise) {
        System.out.println("Closing ..");
        ctx.close(promise);
    }
}
```

<font>构建管道</font>

一个用户应该在一个管道中有一个或多个ChannelHandlers来接收I/O事件(如读)和请求I/O操作(如写和关闭)。例如，一个典型的服务器在每个通道的管道中会有以下处理程序，但实际使用的时间可能会根据协议和业务逻辑的复杂性和特征而不同:

1. 协议解码器(Protocol Decoder) - 将二进制数据(如ByteBuf)转换为Java对象。
2. 协议编码器(Protocol Encoder) - 将Java对象转换为二进制数据。
3. 业务逻辑处理程序(Business Logic Handler) - 执行实际的业务逻辑(例如数据库访问)。

可以用下面的例子来表示：

```java
static final EventExecutorGroup group = new DefaultEventExecutorGroup(16);
...

ChannelPipeline pipeline = ch.pipeline();

pipeline.addLast("decoder", new MyProtocolDecoder());
pipeline.addLast("encoder", new MyProtocolEncoder());

// 告诉管道在与I/O线程不同的线程中运行MyBusinessLogicHandler的事件处理程序方法，
// 以便I/O线程不会被耗时的任务阻塞。
// 如果您的业务逻辑是完全异步的或完成得非常快，则不需要指定组。
pipeline.addLast(group, "handler", new MyBusinessLogicHandler());
```

请注意，当使用DefaultEventLoopGroup时，它将从EventLoop中卸载操作，它仍然会按照ChannelHandlerContext以连续方式处理任务，从而保证排序。由于排序，它可能仍然成为一个瓶颈。如果您的用例不需要排序，您可能需要考虑使用UnorderedThreadPoolEventExecutor来最大化任务执行的并行性。

<font>线程安全</font>

ChannelHandler可以在任何时候添加或删除，因为ChannelPipeline是线程安全的。例如，您可以在将要交换敏感信息时插入加密处理程序，并在交换后删除它。

ChannelPipeline接口源码：

```java
public interface ChannelPipeline
        extends ChannelInboundInvoker, ChannelOutboundInvoker, Iterable<Entry<String, ChannelHandler>> {
    ChannelPipeline addFirst(String name, ChannelHandler handler);
    ChannelPipeline addFirst(EventExecutorGroup group, String name, ChannelHandler handler);
    ChannelPipeline addLast(String name, ChannelHandler handler);
    ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler);
    ChannelPipeline addBefore(String baseName, String name, ChannelHandler handler);
    ChannelPipeline addBefore(EventExecutorGroup group, String baseName, String name, ChannelHandler handler);
    ChannelPipeline addAfter(String baseName, String name, ChannelHandler handler);
    ChannelPipeline addAfter(EventExecutorGroup group, String baseName, String name, ChannelHandler handler);
    ChannelPipeline addFirst(ChannelHandler... handlers);
    ChannelPipeline addFirst(EventExecutorGroup group, ChannelHandler... handlers);
    ChannelPipeline addLast(ChannelHandler... handlers);
    ChannelPipeline addLast(EventExecutorGroup group, ChannelHandler... handlers);
    ChannelPipeline remove(ChannelHandler handler);
    ChannelHandler remove(String name);
    <T extends ChannelHandler> T remove(Class<T> handlerType);
    ChannelHandler removeFirst();
    ChannelHandler removeLast();
    ChannelPipeline replace(ChannelHandler oldHandler, String newName, ChannelHandler newHandler);
    ChannelHandler replace(String oldName, String newName, ChannelHandler newHandler);
    <T extends ChannelHandler> T replace(Class<T> oldHandlerType, String newName,
                                         ChannelHandler newHandler);
    ChannelHandler first();
    ChannelHandlerContext firstContext();
    ChannelHandler last();
    ChannelHandlerContext lastContext();
    ChannelHandler get(String name);
    <T extends ChannelHandler> T get(Class<T> handlerType);
    ChannelHandlerContext context(ChannelHandler handler);
    ChannelHandlerContext context(String name);
    ChannelHandlerContext context(Class<? extends ChannelHandler> handlerType);
    Channel channel();
    List<String> names();
    Map<String, ChannelHandler> toMap();
    @Override
    ChannelPipeline fireChannelRegistered();
    @Override
    ChannelPipeline fireChannelUnregistered();
    @Override
    ChannelPipeline fireChannelActive();
    @Override
    ChannelPipeline fireChannelInactive();
    @Override
    ChannelPipeline fireExceptionCaught(Throwable cause);
    @Override
    ChannelPipeline fireUserEventTriggered(Object event);
    @Override
    ChannelPipeline fireChannelRead(Object msg);
    @Override
    ChannelPipeline fireChannelReadComplete();
    @Override
    ChannelPipeline fireChannelWritabilityChanged();
    @Override
    ChannelPipeline flush();
}
```

## DefaultChannelPipeline

ChannelPipeline只有一个默认实现，即DefaultChannelPipeline：

```java
public class DefaultChannelPipeline implements ChannelPipeline {
	......
	// 头节点名称
    private static final String HEAD_NAME = generateName0(HeadContext.class);
    // 尾节点名称
    private static final String TAIL_NAME = generateName0(TailContext.class);
    // 名称缓存，缓存是一个弱引用，每次GC时都会被回收。
    private static final FastThreadLocal<Map<Class<?>, String>> nameCaches =
            new FastThreadLocal<Map<Class<?>, String>>() {
        @Override
        protected Map<Class<?>, String> initialValue() {
            return new WeakHashMap<Class<?>, String>();
        }
    };

    private static final AtomicReferenceFieldUpdater<DefaultChannelPipeline, MessageSizeEstimator.Handle> ESTIMATOR =
            AtomicReferenceFieldUpdater.newUpdater(
                    DefaultChannelPipeline.class, MessageSizeEstimator.Handle.class, "estimatorHandle");
    final AbstractChannelHandlerContext head;
    final AbstractChannelHandlerContext tail;

    private final Channel channel;
    private final ChannelFuture succeededFuture;
    private final VoidChannelPromise voidPromise;
    private final boolean touch = ResourceLeakDetector.isEnabled();

    private Map<EventExecutorGroup, EventExecutor> childExecutors;
    private volatile MessageSizeEstimator.Handle estimatorHandle;
    private boolean firstRegistration = true;
    // 一个单向链表，用于在通道未注册到EventLoop上时，执行添加，删除，
    // 替换时存储需要执行的回调任务的单向链表。
    private PendingHandlerCallback pendingHandlerCallbackHead;
	// 注册了AbstractChannel后设置为true。一旦设置为true，该值将永远不会改变。
    private boolean registered;
```

### 添加ChannelHandler

对添加ChannelHandler的操作包含从头部、尾部、之前、之后，单个、多个等不同添加方式。

```java
ChannelPipeline addFirst(String name, ChannelHandler handler);
ChannelPipeline addFirst(EventExecutorGroup group, String name, ChannelHandler handler);
ChannelPipeline addLast(String name, ChannelHandler handler);
ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler);
ChannelPipeline addBefore(String baseName, String name, ChannelHandler handler);
ChannelPipeline addBefore(EventExecutorGroup group, String baseName, String name, ChannelHandler handler);
ChannelPipeline addAfter(String baseName, String name, ChannelHandler handler);
ChannelPipeline addAfter(EventExecutorGroup group, String baseName, String name, ChannelHandler handler);
ChannelPipeline addFirst(ChannelHandler... handlers);
ChannelPipeline addFirst(EventExecutorGroup group, ChannelHandler... handlers);
ChannelPipeline addLast(ChannelHandler... handlers);
ChannelPipeline addLast(EventExecutorGroup group, ChannelHandler... handlers);
```

**添加ChannelHandler源码分析**

以下仅仅以addFirst进行分析，至于其他实现，都大同小异，不在做过多阐述。

```java
@Override
public final ChannelPipeline addFirst(String name, ChannelHandler handler) {
    return addFirst(null, name, handler);
}

@Override
public final ChannelPipeline addFirst(EventExecutorGroup group, String name, ChannelHandler handler) {
    final AbstractChannelHandlerContext newCtx;
    synchronized (this) {
        checkMultiplicity(handler);
        name = filterName(name, handler);

        newCtx = newContext(group, name, handler);

        addFirst0(newCtx);

        // 如果registered为false，这意味着通道还没有在eventLoop上注册。
        // 在本例中，我们将上下文添加到管道，并添加一个任务，
        // 该任务将在通道注册后调用ChannelHandler.handlerAdded(...)。
        if (!registered) {
            newCtx.setAddPending();
            callHandlerCallbackLater(newCtx, true);
            return this;
        }

        EventExecutor executor = newCtx.executor();
        if (!executor.inEventLoop()) {
            callHandlerAddedInEventLoop(newCtx, executor);
            return this;
        }
    }
    callHandlerAdded0(newCtx);
    return this;
}


```

首先调用filterName过滤名称，即名称不能相同。如果没有指定名称，在filterName中，将会生成一个名称。如果存在相同的名称，则以名称+数字编号（递增）的方式拼接成新的名称，并返回。

```java
private String filterName(String name, ChannelHandler handler) {
    if (name == null) {
        return generateName(handler);
    }
    checkDuplicateName(name);
    return name;
}

private String generateName(ChannelHandler handler) {
    Map<Class<?>, String> cache = nameCaches.get();
    Class<?> handlerType = handler.getClass();
    String name = cache.get(handlerType);
    if (name == null) {
        name = generateName0(handlerType);
        cache.put(handlerType, name);
    }

    // 用户不太可能放置多个相同类型的处理程序，但要确保避免任何名称冲突。
    // 注意，我们没有缓存这里生成的名称。
    if (context0(name) != null) {
        String baseName = name.substring(0, name.length() - 1); // 去掉末尾的'0'。
        for (int i = 1;; i ++) {
            String newName = baseName + i;
            if (context0(newName) == null) {
                name = newName;
                break;
            }
        }
    }
    return name;
}

private static String generateName0(Class<?> handlerType) {
    return StringUtil.simpleClassName(handlerType) + "#0";
}

private AbstractChannelHandlerContext context0(String name) {
    AbstractChannelHandlerContext context = head.next;
    while (context != tail) {
        if (context.name().equals(name)) {
            return context;
        }
        context = context.next;
    }
    return null;
}
```

然后调用newContext创建一个新的ChannelHandlerContext实例。

```java
private AbstractChannelHandlerContext newContext(EventExecutorGroup group, String name, ChannelHandler handler) {
    return new DefaultChannelHandlerContext(this, childExecutor(group), name, handler);
}
```

最后调用addFirst0将ChannelHandlerContext添加到ChannelPipeline的双向链表中。根据调用的函数不同，其插入到链表的位置不同。如下addFirst0就是将其指定的ChannelHandlerContext添加到链表的头部。

```java
private void addFirst0(AbstractChannelHandlerContext newCtx) {
    AbstractChannelHandlerContext nextCtx = head.next;
    newCtx.prev = head;
    newCtx.next = nextCtx;
    head.next = newCtx;
    nextCtx.prev = newCtx;
}
```

接下来判断通道是否已经在EventLoop上注册，因为在添加ChannelHandler的时候，此时可用通道还未在EventLoop上注册，因此无法回调ChannelHandler的handlerAdded(ChannelHandlerContext)方法，为此，有必要将这个回调操作以任务的形式存储起来，待通道向EventLoop注册时回调。

存储操作如下：

```java
private void callHandlerCallbackLater(AbstractChannelHandlerContext ctx, boolean added) {
    assert !registered;

    PendingHandlerCallback task = added ? new PendingHandlerAddedTask(ctx) : new PendingHandlerRemovedTask(ctx);
    PendingHandlerCallback pending = pendingHandlerCallbackHead;
    if (pending == null) {
        pendingHandlerCallbackHead = task;
    } else {
        // Find the tail of the linked-list.
        while (pending.next != null) {
            pending = pending.next;
        }
        pending.next = task;
    }
}
```

如果是添加操作，added参数为true；如果是删除操作，added参数为false。然后将其添加到单链表pendingHandlerCallbackHead的末尾。最后return this。

如果已经通道已经注册到EventLoop了，还需要分两种情况：

1. 添加ChannelHandler操作是在当前EventLoop线程。

   在这种情况下就需要马上回调ChannelHandler#handlerAdded(ChannelHandlerContext)方法。

   ```java
   private void callHandlerAdded0(final AbstractChannelHandlerContext ctx) {
       try {
           ctx.callHandlerAdded();
       } catch (Throwable t) {
           boolean removed = false;
           try {
               atomicRemoveFromHandlerList(ctx);
               ctx.callHandlerRemoved();
               removed = true;
           } catch (Throwable t2) {
               if (logger.isWarnEnabled()) {
                   logger.warn("Failed to remove a handler: " + ctx.name(), t2);
               }
           }
   
           if (removed) {
               fireExceptionCaught(new ChannelPipelineException(
                   ctx.handler().getClass().getName() +
                   ".handlerAdded() has thrown an exception; removed.", t));
           } else {
               fireExceptionCaught(new ChannelPipelineException(
                   ctx.handler().getClass().getName() +
                   ".handlerAdded() has thrown an exception; also failed to remove.", t));
           }
       }
   }
   ```

2. 添加ChannelHandler操作不是在当前EventLoop线程。

   在这种情况下以普通任务的形式交由事件循环执行。

   ```java
   private void callHandlerAddedInEventLoop(final AbstractChannelHandlerContext newCtx, EventExecutor executor) {
       newCtx.setAddPending();
       executor.execute(new Runnable() {
           @Override
           public void run() {
               callHandlerAdded0(newCtx);
           }
       });
   }
   ```

### 删除ChannelHandler

删除ChannelHandler操作按头部，尾部，类型，名称，Handler等方式删除。

```java
//从此管道移除指定的ChannelHandler。
ChannelPipeline remove(ChannelHandler handler);
//从此管道移除具有指定名称的ChannelHandler。
ChannelHandler remove(String name);
//从此管道移除指定类型的ChannelHandler。
<T extends ChannelHandler> T remove(Class<T> handlerType);
//删除此管道中的第一个ChannelHandler。
ChannelHandler removeFirst();
//删除此管道中的最后一个ChannelHandler。
ChannelHandler removeLast();
```

源码分析

通过remove(String name)进行分析，实现如下：

```java
@Override
public final ChannelHandler remove(String name) {
    return remove(getContextOrDie(name)).handler();
}
```

首先是通过getContextOrDie()函数获取到需要删除的ChannelHandler，然后交由私有方法remove()操作：

```java
private AbstractChannelHandlerContext remove(final AbstractChannelHandlerContext ctx) {
    assert ctx != head && ctx != tail;

    synchronized (this) {
        atomicRemoveFromHandlerList(ctx);

        // 如果注册为false，则意味着通道还没有在事件循环中注册。
        // 在本例中，我们从管道中删除上下文，并添加一个任务，该任务将在通道注册后调用
        // ChannelHandler.handlerRemoved(…)。
        if (!registered) {
            callHandlerCallbackLater(ctx, false);
            return ctx;
        }

        EventExecutor executor = ctx.executor();
        if (!executor.inEventLoop()) {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    callHandlerRemoved0(ctx);
                }
            });
            return ctx;
        }
    }
    callHandlerRemoved0(ctx);
    return ctx;
}
```

删除操作就是调用atomicRemoveFromHandlerList方法，从链表中断开对被删除ChannelHandler的链接。

```java
private synchronized void atomicRemoveFromHandlerList(AbstractChannelHandlerContext ctx) {
    AbstractChannelHandlerContext prev = ctx.prev;
    AbstractChannelHandlerContext next = ctx.next;
    prev.next = next;
    next.prev = prev;
}
```

删除操作跟添加操作一样，也需要考虑通道是否已经在EventLoop上注册。以及，如果已经注册了，删除操作是在EventLoop线程上删除还是不是的情况。然后执行模式都是一样的。

另外，DefaultChannelPipeline还提供了如下三个删除方法：如果存在，则删除，反之，则返回NULL。而实现接口的删除方法是如果不存在，就会抛出NoSuchElementException异常。

```java
public final <T extends ChannelHandler> T removeIfExists(String name) {
    return removeIfExists(context(name));
}

public final <T extends ChannelHandler> T removeIfExists(Class<T> handlerType) {
    return removeIfExists(context(handlerType));
}

public final <T extends ChannelHandler> T removeIfExists(ChannelHandler handler) {
    return removeIfExists(context(handler));
}

@SuppressWarnings("unchecked")
private <T extends ChannelHandler> T removeIfExists(ChannelHandlerContext ctx) {
    if (ctx == null) {
        return null;
    }
    return (T) remove((AbstractChannelHandlerContext) ctx).handler();
}
```

### 替换ChannelHandler

替换ChannelHandler操作按类型，名称，Handler等方式替换。

```java
//用此管道中的新处理程序替换指定的ChannelHandler。
ChannelPipeline replace(ChannelHandler oldHandler, String newName, ChannelHandler newHandler);
//将指定名称的ChannelHandler替换为此管道中的新处理程序。
ChannelHandler replace(String oldName, String newName, ChannelHandler newHandler);
//用此管道中的新处理程序替换指定类型的ChannelHandler。
<T extends ChannelHandler> T replace(Class<T> oldHandlerType, String newName,
                                     ChannelHandler newHandler);
```

源码分析

```java
@Override
public final ChannelHandler replace(String oldName, String newName, ChannelHandler newHandler) {
    return replace(getContextOrDie(oldName), newName, newHandler);
}

private ChannelHandler replace(
    final AbstractChannelHandlerContext ctx, String newName, ChannelHandler newHandler) {
    assert ctx != head && ctx != tail;

    final AbstractChannelHandlerContext newCtx;
    synchronized (this) {
        checkMultiplicity(newHandler);
        if (newName == null) {
            // 生产一个新的ChannelHandler的名称
            newName = generateName(newHandler);
        } else {
            //制定了新名称，检查名称是否重复
            boolean sameName = ctx.name().equals(newName);
            if (!sameName) {
                checkDuplicateName(newName);
            }
        }
		//创建一个新的ChannelHandlerContext
        newCtx = newContext(ctx.executor, newName, newHandler);
		//从链表中移除旧元素，添加新元素
        replace0(ctx, newCtx);
		//替换操作需要回调ChannelHandler.handlerAdded(...)和
        //ChannelHandler.handlerRemoved(...)，因此，同样需要考虑
        //通道是否已经在Eventloop注册。
        if (!registered) {
            callHandlerCallbackLater(newCtx, true);
            callHandlerCallbackLater(ctx, false);
            return ctx.handler();
        }
        // 考虑是否是在当前Eventloop线程上执行替换。
        EventExecutor executor = ctx.executor();
        if (!executor.inEventLoop()) {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    // 首先调用newHandler.handlerAdded()
                    //(即在调用oldHandler.handlerRemoved()之前)，因为
                    // callHandlerRemoved()会在newHandler上触发channelRead()或flush()，
                    // 而这些事件处理程序必须在handlerAdded()之后调用。
                    callHandlerAdded0(newCtx);
                    callHandlerRemoved0(ctx);
                }
            });
            return ctx.handler();
        }
    }
    callHandlerAdded0(newCtx);
    callHandlerRemoved0(ctx);
    return ctx.handler();
}
```

替换操作也没有太复杂的。只是同样需要考虑通道是否已经在EventLoop上注册。以及，如果已经注册了，删除操作是在EventLoop线程上删除还是不是的情况。另外一点需要注意的是，替换操作首先调用newHandler.handlerAdded()（即在调用oldHandler.handlerRemoved()之前），因为callHandlerRemoved()会在newHandler上触发channelRead()或flush()，而这些事件处理程序必须在handlerAdded()之后调用。

### 获取ChannelHandler和ChannelHandlerContext

同样，按头部，尾部，类型，名称，Handler等方式获取。

```java
ChannelHandler first();
ChannelHandlerContext firstContext();
ChannelHandler last();
ChannelHandlerContext lastContext();
ChannelHandler get(String name);
<T extends ChannelHandler> T get(Class<T> handlerType);
ChannelHandlerContext context(ChannelHandler handler);
ChannelHandlerContext context(String name);
ChannelHandlerContext context(Class<? extends ChannelHandler> handlerType);
```

源码分析

以get(String name)为例进行分析。

```java
@Override
public final ChannelHandler get(String name) {
    ChannelHandlerContext ctx = context(name);
    if (ctx == null) {
        return null;
    } else {
        return ctx.handler();
    }
}

@Override
public final ChannelHandlerContext context(String name) {
    return context0(ObjectUtil.checkNotNull(name, "name"));
}

private AbstractChannelHandlerContext context0(String name) {
    AbstractChannelHandlerContext context = head.next;
    while (context != tail) {
        if (context.name().equals(name)) {
            return context;
        }
        context = context.next;
    }
    return null;
}
```

根据给定的名称依次迭代链表元素，并对比名称，如果相同则返回，反之没有相同，则返回NULL。

至于其他的获取方法都大同小异，以轮询的方式进行对比，对比成功，则获取成功。

### 通道未注册到EventLoop

前面在分析添加，删除，替换操作时，说到要考虑通道是否注册到EventLoop上的情况，如果通道未注册到EventLoop上，那么就需要将对handlerAdded()/handlerRemoved()方法的回调以任务的形式存储起来，待通道向EventLoop注册时回调。

这个操作是在DefaultChannelPipeline内维护了一个单向链表PendingHandlerCallback。成员变量pendingHandlerCallbackHead仅仅是表头。

```java
public class DefaultChannelPipeline implements ChannelPipeline {
	......
    private PendingHandlerCallback pendingHandlerCallbackHead;
    ......
}
```

添加该单向链表的操作是通过callHandlerCallbackLater()方法完成（添加ChannelHandler小节已有描述）。

当通道注册到EventLoop上时，回调操作是由头节点的channelRegistered方法触发。

```java
final class HeadContext extends AbstractChannelHandlerContext
    implements ChannelOutboundHandler, ChannelInboundHandler {
	......
    @Override
    public void channelRegistered(ChannelHandlerContext ctx) {
        invokeHandlerAddedIfNeeded();
        ctx.fireChannelRegistered();
    }
    ......
}
```

下一步调用invokeHandlerAddedIfNeeded()方法：

```java
final void invokeHandlerAddedIfNeeded() {
    assert channel.eventLoop().inEventLoop();
    if (firstRegistration) {
        firstRegistration = false;
        callHandlerAddedForAllHandlers();
    }
}
```

真正的回调在callHandlerAddedForAllHandlers()方法中：

```java
private void callHandlerAddedForAllHandlers() {
    final PendingHandlerCallback pendingHandlerCallbackHead;
    synchronized (this) {
        assert !registered;
        // 标记该Channel本身已注册。
        registered = true;
        pendingHandlerCallbackHead = this.pendingHandlerCallbackHead;
        // 这是为NULL，这样可以被GC
        this.pendingHandlerCallbackHead = null;
    }
    // 这必须发生在synchronized(…)块之外，否则handlerAdded(…)可能在持有锁时被调用，
    // 因此如果handlerAdded(…)试图从EventLoop外部添加另一个处理程序，就会产生死锁。
    PendingHandlerCallback task = pendingHandlerCallbackHead;
    while (task != null) {
        task.execute();
        task = task.next;
    }
}
```

回调操作很简单：迭代单向链表，依次执行任务的execute()方法。

### 头结点/尾节点

在ChannelHandler双向链表中，头结点和尾节点的类型固定为io.netty.channel.DefaultChannelPipeline.HeadContext和io.netty.channel.DefaultChannelPipeline.TailContext，对于HeadContext和TailContext，它们主要是作为链表中头结点和尾节点的标记位。

具体而言，对于TailContext，会执行一些初始化回调，例如回调handlerAdded()或handlerRemoved()方法，同时触发下一个ChannelHandler的调用。对于TailContext，会调用DefaultChannelPipeline中的onUnhandled*系列方法，做一些资源的收尾工作。

```java
public class DefaultChannelPipeline implements ChannelPipeline {
    ......
    final class HeadContext extends AbstractChannelHandlerContext
        implements ChannelOutboundHandler, ChannelInboundHandler {
        ......
    }
    final class TailContext extends AbstractChannelHandlerContext implements ChannelInboundHandler {
        ......
    }
    ......
}
```

如上代码所示，出站和入栈的时候都会经过HeadContext，而TailContext，只有在入栈的时候才会经过。

## ChannelHandlerContext

ChannelHandlerContext是DefaultChannelPipeline中双向链表的元素类型，至于ChannelHandler则是包装在ChannelHandlerContext之中。

ChannelHandlerContext类继承结构如下图所示：

![]()

### ChannelHandlerContext

```java
public interface ChannelHandlerContext extends AttributeMap, ChannelInboundInvoker, ChannelOutboundInvoker {
    Channel channel();
    EventExecutor executor();
    String name();
    ChannelHandler handler();
    boolean isRemoved();
    @Override
    ChannelHandlerContext fireChannelRegistered();
    @Override
    ChannelHandlerContext fireChannelUnregistered();
    @Override
    ChannelHandlerContext fireChannelActive();
    @Override
    ChannelHandlerContext fireChannelInactive();
    @Override
    ChannelHandlerContext fireExceptionCaught(Throwable cause);
    @Override
    ChannelHandlerContext fireUserEventTriggered(Object evt);
    @Override
    ChannelHandlerContext fireChannelRead(Object msg);
    @Override
    ChannelHandlerContext fireChannelReadComplete();
    @Override
    ChannelHandlerContext fireChannelWritabilityChanged();
    @Override
    ChannelHandlerContext read();
    @Override
    ChannelHandlerContext flush();
    ChannelPipeline pipeline();
    ByteBufAllocator alloc();
    @Deprecated
    @Override
    <T> Attribute<T> attr(AttributeKey<T> key);
    @Deprecated
    @Override
    <T> boolean hasAttr(AttributeKey<T> key);
}
```

API如下：

- `Channel channel()`: 返回绑定到ChannelHandlerContext的Channel。
- `EventExecutor executor()`: 返回用于执行任意任务的EventExecutor。
- `String name()`: ChannelHandlerContext的唯一名称。将ChannelHandler添加到ChannelPipeline时使用了该名称。这个名称还可以用于从ChannelPipeline访问已注册的ChannelHandler。
- `ChannelHandler handler()`: 绑定到ChannelHandlerContext的ChannelHandler。
- `boolean isRemoved()`: 如果属于此上下文的ChannelHandler从ChannelPipeline中移除，则返回true。注意，这个方法只意味着在EventLoop中从with调用。Note that this method is only meant to be called from with in the EventLoop.
- `ChannelPipeline pipeline()`: 返回指定的ChannelPipeline。
- `ByteBufAllocator alloc()`: 返回分配的ByteBufAllocator，它将用于分配bytebuf。

### DefaultChannelHandlerContext

DefaultChannelHandlerContext是一个package访问权限的类，内部持有了一个对ChannelHandler的引用。

```java
final class DefaultChannelHandlerContext extends AbstractChannelHandlerContext {

    private final ChannelHandler handler;

    DefaultChannelHandlerContext(
            DefaultChannelPipeline pipeline, EventExecutor executor, String name, ChannelHandler handler) {
        super(pipeline, executor, name, handler.getClass());
        this.handler = handler;
    }

    @Override
    public ChannelHandler handler() {
        return handler;
    }
}
```

### AbstractChannelHandlerContext



```java
abstract class AbstractChannelHandlerContext implements ChannelHandlerContext, ResourceLeakHint {
	......
    // 指向前驱节点的指针
    volatile AbstractChannelHandlerContext next;
    // 指向后驱节点的指针
    volatile AbstractChannelHandlerContext prev;

    private static final AtomicIntegerFieldUpdater<AbstractChannelHandlerContext> HANDLER_STATE_UPDATER =
            AtomicIntegerFieldUpdater.newUpdater(AbstractChannelHandlerContext.class, "handlerState");

    /**
     * {@link ChannelHandler#handlerAdded(ChannelHandlerContext)} is about to be called.
     */
    private static final int ADD_PENDING = 1;
    /**
     * {@link ChannelHandler#handlerAdded(ChannelHandlerContext)} was called.
     */
    private static final int ADD_COMPLETE = 2;
    /**
     * {@link ChannelHandler#handlerRemoved(ChannelHandlerContext)} was called.
     */
    private static final int REMOVE_COMPLETE = 3;
    /**
     * ChannelHandler.handlerAdded(ChannelHandlerContext)和     
     * ChannelHandler.handlerRemoved(ChannelHandlerContext)都没有被调用。
     */
    private static final int INIT = 0;
	// 对DefaultChannelPipeline的引用
    private final DefaultChannelPipeline pipeline;
    // ChannelHandler的名称
    private final String name;
    private final boolean ordered;
    private final int executionMask;

    // 如果不应该使用子执行程序，则将设置为空，否则将设置为子执行程序。
    final EventExecutor executor;
    private ChannelFuture succeededFuture;

    // 惰性实例化任务，用于将事件触发到具有不同执行器的处理程序。
    // 没有必要让它变得不稳定，因为在更糟的情况下，它只会创建更多的实例。
    private Tasks invokeTasks;

    private volatile int handlerState = INIT;
```