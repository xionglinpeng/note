# Scalable IO in Java  

<div align="center">Doug Lea</div>
<div align="center">纽约州立大学奥斯威戈分校</div>
<div align="center">dl@cs.oswego.edu</div>
<div align="center">http://gee.cs.oswego.edu</div>
## 大纲

- 可伸缩网络服务

- 事件驱动处理

- 反应堆模式
  - 基础版本
  - 多线程版本
  - 其他变体

-  `java.nio`非阻塞IO API的预排

## 网络服务

-  Web服务，分布式对象等

- 大多数都有相同的基本结构：
  <font style="background-color:#e91e64">Read request</font> → <font style="background-color:#e91e64">Decode request</font> → <font style="background-color:#e91e64">Process service</font> → <font style="background-color:#e91e64">Encode reply</font> → <font style="background-color:#e91e64">Send reply</font>

- 但每个步骤的性质和成本不同
  XML解析，文件传输，网页
  生成、计算服务……

## 经典的服务设计

![image-20220624142742621](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220624142742621.png)

每个处理程序可以在自己的线程中启动。

### 经典ServerSocket循环

```java
public class Server implements Runnable {
    @Override
    public void run() {
        try {
            ServerSocket ss = new ServerSocket(8080);
            while (!Thread.interrupted())
                new Thread(new Handler(ss.accept())).start();
            //或者单个线程，或者线程池
        } catch (Exception e) { /* ... */ }
    }

    @AllArgsConstructor
    static class Handler implements Runnable {
        private final Socket socket;
        @Override
        public void run() {
            try {
                byte[] input = new byte[1024];
                socket.getInputStream().read(input);
                byte[] output = process(input);
                socket.getOutputStream().write(output);
            } catch (Exception e) { /* ... */ }
        }

        private byte[] process(byte[] data) {/* processor ... */}
    }
}
```

Note: 代码示例中省略了大多数异常处理。

## 可伸缩性目标

- 在不断增加的负载(更多的客户机)下的优雅降级。
  
- 不断提高资源(CPU、内存、磁盘、带宽)。
  
- 还要满足可用性和性能目标：短延时、满足高峰需求、可调的服务质量。
  
- 分治通常是实现任何可伸缩性目标的最佳方法。

## 分而治之

- 将处理过程分成小任务，每个任务执行一个没有阻塞的操作。
  
- 当每个任务被启用时执行它。这里，IO事件通常充当触发器。
  
  ![image-20220624143338120](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220624143338120.png)
  
- java.nio支持的基本机制：非阻塞读和写、分派与感知IO事件相关的任务。
  
- 无穷无尽的变化可能。一系列事件驱动的设计。

## 事件驱动设计

- 通常比替代方案更有效
  
  - 更少的资源：通常不需要每个客户端使用一个线程。
  
  - 减少开销：更少的上下文切换，通常更少的锁定。但发送速度可能会更慢。
  - 必须手动将操作绑定到事件。
  
- 通常更难编程
  
  - 必须分解成简单的非阻塞操作。
  
  	类似于GUI事件驱动的操作。
  
  	不能消除所有阻塞：GC、页面错误等。
  
  - 必须跟踪服务的逻辑状态。

## 背景:AWT中的事件 

![image-20220624141804567](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220624141804567.png)

事件驱动的IO使用类似的思想，但采用不同的设计。

## Reactor模式

- Reactor通过调度适当的处理程序来响应IO事件。
  
  类似于AWT线程
  
- 处理程序执行非阻塞操作。

  类似于AWT ActionListeners

- 通过将处理程序绑定到事件进行管理。

  类似于AWT addActionListener

- 参见Schmidt等人，面向模式的软件体系结构，第2卷(POSA2)。

  还有Richard Stevens的网络书籍，Matt Welsh的SEDA框架等

### 基本的Reactor设计

![image-20220624145051596](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220624145051596.png)

单线程版本

#### java.nio支持

- `Channels`
  
  连接到文件，套接字等，支持非阻塞读取。
  
- `Buffers`
  
  可以由channel直接读取或写入的类数组对象。
  
- `Selectors`
  
  告诉一组通道中哪些有IO事件。
  
- `SelectionKeys`
  
  维护IO事件状态和绑定。

#### Reactor 1: Setup  

```java
public class Reactor implements Runnable {
    private final Selector selector;
    private final ServerSocketChannel serverSocket;
    public Reactor(int port) throws IOException {
        selector = Selector.open();
        serverSocket = ServerSocketChannel.open();
        serverSocket.configureBlocking(false);
        serverSocket.socket().bind(new InetSocketAddress(port));
        serverSocket.register(selector, SelectionKey.OP_ACCEPT, new Acceptor());
    }
}
```

#### Reactor 2: Dispatch Loop 

```java
@Override
public void run() {
    try {
        while (!Thread.interrupted()) {
            selector.select();
            Set<SelectionKey> sks = selector.selectedKeys();
            for (SelectionKey sk : sks)
                dispatch(sk);
            sks.clear();
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}

public void dispatch(SelectionKey sk) {
    Runnable r = (Runnable) sk.attachment();
    if (r != null)
        r.run();
}
```

#### Reactor 3: Acceptor  

```java
class Acceptor implements Runnable {
    @Override
    public void run() {
        try {
            SocketChannel sc = serverSocket.accept();
            if (sc != null) new ReactorHandler(sc, selector);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

![image-20220624145228739](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220624145228739.png)

#### Reactor 4: Handler setup  

```java
public class Handler implements Runnable {
    private final SocketChannel socket;
    private final SelectionKey sk;
    private static final byte READING = 0, WRITING = 1;
    private byte state = READING;

    public Handler(SocketChannel socket, Selector selector) 
        												throws IOException {
        this.socket = socket;
        socket.configureBlocking(false);
        sk = socket.register(selector, 0);
        //将Handler作为回调对象 → ReactorHandler#run()
        sk.attach(this);
        sk.interestOps(SelectionKey.OP_READ);
        selector.wakeup();
    }

    private boolean inputIsComplete() throws IOException {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        int len;
        while ((len = socket.read(buffer)) != 0) {
            buffer.flip();
            System.out.print(new String(buffer.array(), 0, len));
            buffer.clear();
        }
        return true;
    }

    private boolean outputIsComplete() throws IOException {
        socket.write(ByteBuffer.wrap("我收到啦".getBytes()));
        return true;
    }

    private void process() {}
}
```

#### Reactor 5: Request handling  

```java
@Override
public void run() {
    try {
        if (state == READING) read();
        if (state == WRITING) write();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

private void read() throws IOException {
    if (inputIsComplete()) {
        process();
        state = WRITING;
        sk.interestOps(SelectionKey.OP_WRITE);
    }
}

private void write() throws IOException {
    if (outputIsComplete())
        sk.cancel();
}
```

#### Per-State Handlers  

GoF状态-对象模式的简单使用。在`run()`中完成读取操作，然后将写操作处理程序重新绑定为`SelectionKey`的附件。

```java
@Override
public void run() {
    try {
        if (inputIsComplete()) {
            process();
            sk.attach(new Sender());
            sk.interestOps(SelectionKey.OP_WRITE);
            sk.selector().wakeup();
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}

class Sender implements Runnable {
    @Override
    public void run() {
        try {
            if (outputIsComplete())
                sk.cancel();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 多线程设计 

- 策略性地添加线程以实现可伸缩性
  主要适用于多处理器
- 工作线程
  Reactor会迅速触发处理程序
      处理器的处理减慢了Reactor的速度
  将非IO处理交给其他线程
- 多个Reactor线程
  Reactor线程在进行IO时可能会饱和
  将负载分配给其他Reactor
      负载均衡，以匹配CPU和IO速率

#### 工作线程池  

- 卸载非io处理，以加快Reactor线程
  类似于POSA2 Proactor设计

- 比将计算绑定的处理重做为事件驱动的形式更简单
  仍然应该是纯的非阻塞计算
      足以超过开销的处理

- 但是很难与IO重叠处理
  最好的时候可以先把所有的输入读入缓冲区

- 使用线程池可以调优和控制
  通常需要比客户端少得多的线程

#### 工作线程池

![image-20220624150845845](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220624150845845.png)

#### 带有线程池的处理程序

```java
public class ReactorHandler implements Runnable {

    private static final byte PROCESSING = 2;
    private static final ExecutorService pool = Executors.newFixedThreadPool(4);

    private synchronized void read() throws IOException {
        if (inputIsComplete()) {
            state = PROCESSING;
            pool.execute(new Processor());
        }
    }

    private synchronized void processAndHandOff() {
        process();
        state = WRITING;
        sk.interestOps(SelectionKey.OP_WRITE);
        sk.selector().wakeup();
    }

    class Processor implements Runnable {
        @Override
        public synchronized void run() {
            processAndHandOff();
        }
    }
}
```

#### 协调任务

- hand-offs

  每个任务启用、触发或调用下一个任务通常速度最快，但也可能很脆弱

- 每个处理器分派的回调(Callbacks to per-handler dispatcher )
  设置状态(state)，附件(attachment)等
  GoF中介模式的变体

- Queues
  例如，跨阶段传递缓冲区

- Futures

当每个任务生成一个结果时，协调层位于连接或wait/notify之上。

#### 使用PooledExecutor  

- 可调的工作线程池

- 主方法execute(Runnable r)

- 控制：
  任务队列的类型(任何通道)
  最大线程数
  最小线程数
  “Warm”和随需应变的线程
  保持活动间隔，直到空闲线程死亡(如有需要，可稍后以新的取代)
  饱和的政策(阻塞，下降，生产者运行等)

### 多个Reactor线程

使用Reactor池
    用于匹配CPU和IO速率
    静态或动态构造
        每个都有自己的选择器，线程，分派循环
    主acceptor分配给其他Reactor。

```java
public class Reactor implements Runnable {
    private final Selector[] selectors = new Selector[2];
    private final ServerSocketChannel serverSocket;

    public Reactor(int port) throws IOException {
        for (int i = 0; i < selectors.length; i++)
            selectors[i] = Selector.open();
        serverSocket = ServerSocketChannel.open();
        serverSocket.configureBlocking(false);
        serverSocket.socket().bind(new InetSocketAddress(port));
        serverSocket.register(selectors[0], SelectionKey.OP_ACCEPT, new Acceptor());
    }

    @Override
    public void run() {
        new Thread(new SubReactor()).start();
        select(selectors[0]);
    }

    private void select(Selector selector) {
        try {
            while (!Thread.interrupted()) {
                selector.select();
                Set<SelectionKey> sks = selector.selectedKeys();
                for (SelectionKey sk : sks) {
                    System.out.println(sk.readyOps());
                    dispatch(sk);
                }
                sks.clear();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void dispatch(SelectionKey sk) {
        Runnable r = (Runnable) sk.attachment();
        if (r != null) r.run();
    }
    
    class SubReactor implements Runnable {
        @Override
        public void run() {
            select(selectors[1]);
        }
    }

    class Acceptor implements Runnable {
        @Override
        public synchronized void run() {
            try {
                //主selector负责accept
                SocketChannel sc = serverSocket.accept();
                //子selector负责其他事件
                if (sc != null) new Handler(sc, selectors[1]);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### 使用多个Reactor  

![image-20220624151626387](C:\Users\linpeng.xiong\AppData\Roaming\Typora\typora-user-images\image-20220624151626387.png)

## 使用其它java.nio特性

- 每个Reactor有多个Selector

  将不同的处理程序绑定到不同的IO事件

  可能需要仔细的同步来协调

- 文件传输
  自动文件到网络或网络到文件复制
  
- 内存映射文件
  
  通过缓冲区访问文件
  
- 直接缓冲区
  
  有时实现零拷贝传输
  但是有设置和结束开销
  最适合长时间连接的应用程序

## 基于连接的扩展

- 与单一的服务请求不同
  客户端连接
  客户端发送一系列消息/请求
  客户端断开连接

- 示例
  数据库和事务监听
  多人游戏、聊天等

- 可以扩展基本的网络服务模式
  处理许多相对长期存活的客户端
  跟踪客户端和会话状态(包括删除)
  跨多个主机分配服务

## API预排
Buffer
ByteBuffer (CharBuffer, LongBuffer等没有显示)
Channel
SelectableChannel
SocketChannel
ServerSocketChannel
FileChannel
Selector
SelectionKey  

### Buffer

```java
abstract class Buffer {
    int capacity();
    int position();
    Buffer position(int newPosition);
    int limit();
    Buffer limit(int newLimit);
    Buffer mark();
    Buffer reset();
    Buffer clear();
    Buffer flip();
    Buffer rewind();
    int remaining();
    boolean hasRemaining();
    boolean isReadOnly();
}
```

### ByteBuffer
```java
abstract class ByteBuffer extends Buffer {
    static ByteBuffer allocateDirect(int capacity);
    static ByteBuffer allocate(int capacity);
    static ByteBuffer wrap(byte[] src, int offset, int len);
    static ByteBuffer wrap(byte[] src);
    boolean isDirect();
    ByteOrder order();
    ByteBuffer order(ByteOrder bo);
    ByteBuffer slice();
    ByteBuffer duplicate();
    ByteBuffer compact();
    ByteBuffer asReadOnlyBuffer();
    byte get();
    byte get(int index);
    ByteBuffer get(byte[] dst, int offset, int length);
    ByteBuffer get(byte[] dst);
    ByteBuffer put(byte b);
    ByteBuffer put(int index, byte b);
    ByteBuffer put(byte[] src, int offset, int length);
    ByteBuffer put(ByteBuffer src);
    ByteBuffer put(byte[] src);
    char getChar();
    char getChar(int index);
    ByteBuffer putChar(char value);
    ByteBuffer putChar(int index, char value);
    CharBuffer asCharBuffer();
    short getShort();
    short getShort(int index);
    ByteBuffer putShort(short value);
    ByteBuffer putShort(int index, short value);
    ShortBuffer asShortBuffer();
    int getInt();
    int getInt(int index);
    ByteBuffer putInt(int value);
    ByteBuffer putInt(int index, int value);
    IntBuffer asIntBuffer();
    long getLong();
    long getLong(int index);
    ByteBuffer putLong(long value);
    ByteBuffer putLong(int index, long value);
    LongBuffer asLongBuffer();
    float getFloat();
    float getFloat(int index);
    ByteBuffer putFloat(float value);
    ByteBuffer putFloat(int index, float value);
    FloatBuffer asFloatBuffer();
    double getDouble();
    double getDouble(int index);
    ByteBuffer putDouble(double value);
    ByteBuffer putDouble(int index, double value);
    DoubleBuffer asDoubleBuffer();
}
```

### Channel
```java
interface Channel {
    boolean isOpen();
    void close() throws IOException;
}
interface ReadableByteChannel extends Channel {
    int read(ByteBuffer dst) throws IOException;
}
interface WritableByteChannel extends Channel {
    int write(ByteBuffer src) throws IOException;
}
interface ScatteringByteChannel extends ReadableByteChannel {
    int read(ByteBuffer[] dsts, int offset, int length)
        throws IOException;
    int read(ByteBuffer[] dsts) throws IOException;
}
interface GatheringByteChannel extends WritableByteChannel {
    int write(ByteBuffer[] srcs, int offset, int length)
        throws IOException;
    int write(ByteBuffer[] srcs) throws IOException;
}
```

### SelectableChannel

```java
abstract class SelectableChannel implements Channel {
    int validOps();
    boolean isRegistered();
    SelectionKey keyFor(Selector sel);
    SelectionKey register(Selector sel, int ops) throws ClosedChannelException;
    void configureBlocking(boolean block) throws IOException;
    boolean isBlocking();
    Object blockingLock();
}
```

### SocketChannel

```java
abstract class SocketChannel implements ByteChannel ... {
    static SocketChannel open() throws IOException;
    Socket socket();
    int validOps();
    boolean isConnected();
    boolean isConnectionPending();
    boolean isInputOpen();
    boolean isOutputOpen();
    boolean connect(SocketAddress remote) throws IOException;
    boolean finishConnect() throws IOException;
    void shutdownInput() throws IOException;
    void shutdownOutput() throws IOException;
    int read(ByteBuffer dst) throws IOException;
    int read(ByteBuffer[] dsts, int offset, int length) throws IOException;
    int read(ByteBuffer[] dsts) throws IOException;
    int write(ByteBuffer src) throws IOException;
    int write(ByteBuffer[] srcs, int offset, int length) throws IOException;
    int write(ByteBuffer[] srcs) throws IOException;
}
```

### ServerSocketChannel

```java
abstract class ServerSocketChannel extends ... {
    static ServerSocketChannel open() throws IOException;
    int validOps();
    ServerSocket socket();
    SocketChannel accept() throws IOException;
}
```

### FileChannel

```
abstract class FileChannel implements ... {
    int read(ByteBuffer dst);
    int read(ByteBuffer dst, long position);
    int read(ByteBuffer[] dsts, int offset, int length);
    int read(ByteBuffer[] dsts);
    int write(ByteBuffer src);
    int write(ByteBuffer src, long position);
    int write(ByteBuffer[] srcs, int offset, int length);
    int write(ByteBuffer[] srcs);
    long position();
    void position(long newPosition);
    long size();
    void truncate(long size);
    void force(boolean flushMetaDataToo);
    int transferTo(long position, int count,
    WritableByteChannel dst);
    int transferFrom(ReadableByteChannel src,
    long position, int count);
    FileLock lock(long position, long size, boolean shared);
    FileLock lock();
    FileLock tryLock(long pos, long size, boolean shared);
    FileLock tryLock();
    static final int MAP_RO, MAP_RW, MAP_COW;
    MappedByteBuffer map(int mode, long position, int size);
}
```

NOTE: ALL methods throw IOException

### Selector

```java
abstract class Selector {
    static Selector open() throws IOException;
    Set keys();
    Set selectedKeys();
    int selectNow() throws IOException;
    int select(long timeout) throws IOException;
    int select() throws IOException;
    void wakeup();
    void close() throws IOException;
}
```

### SelectionKey

```java
abstract class SelectionKey {
    static final int OP_READ, OP_WRITE,
    OP_CONNECT, OP_ACCEPT;
    SelectableChannel channel();
    Selector selector();
    boolean isValid();
    void cancel();
    int interestOps();
    void interestOps(int ops);
    int readyOps();
    boolean isReadable();
    boolean isWritable();
    boolean isConnectable();
    boolean isAcceptable();
    Object attach(Object ob);
    Object attachment();
}
```