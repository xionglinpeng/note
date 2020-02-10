# Setvlet

## 异步处理支持

Servlet 3.0之前，一个普通的Servet的主要工作流程大致如下：首先，Servlet接收到请求之后，可能需要对请求携带的数据进行一些预处理；接着，调用业务接口的某些方法，以完成业务处理；最后，根据处理的结果提交响应，Servlet线程结束。其中第二步的业务处理通常是最耗时的，这主要体现在数据库操作，以及其他跨网络调用等，在此过程中，Servlet线程一直处理阻塞状态，直到业务方法执行完毕为止。在处理业务的过程中，Servlet资源一直被占用而得不到释放，对于并发较大的应用，这有可能造成性能的瓶颈。对此，在以前通常是采用是私有的解决方案来提前结束Servlet线程，并及时释放资源。

Setvlet3.0针对这个问题做了开创性的工作，现在通过使用Servlet3.0的异步处理支持，之前的Servlet处理流程可以调整为如下的过程：首先，Servlet接收到请求之后，可能首先需要对请求携带的数据进行一些预处理；接着，Servlet线程将请求转交给一个异步线程来执行业务处理，线程本身返回至容器，此时Servlet还没有生成响应数据，异步线程处理完业务之后，可以直接生成响应数据，或者将请求继续转发给其他Servlet。如此一来，Servlet线程不在是一直处于阻塞状态以等待业务逻辑的处理，而是启动异步线程之后可以立即返回。

异步处理特性可以应用于Servlet和filter两种组件，由于异步处理的工作模式和普通工作模式在实现上有着本质的区别，因此默认情况下，Servlet和filter并没有开启异步处理的特性，如果希望使用该特性，则必须按照如下的方式启用：

1. 对于传统的部署描述文件（web.xml）配置Servlet和filter的情况，Servlet 3.0为`<servlet>`和`<filter>`标签增加了`<async-supported>`子标签，该标签的默认取值为`false`，要启用异步处理支持，则将其设为`true`即可。以Servlet为例，其配置方式如下所示：

   ```xml
   <servlet>
       <servlet-name>async</servlet-name>
       <servlet-class>com.learn.servlets.AsyncServlet</servlet-class>
       <async-supported>true</async-supported>
   </servlet>
   ```

2. 对于使用Servlet 3.0提供的`@WebServlet`和`@WebFilter`注解进行Servlet或filter配置的情况，这两个注解都提供了`asynSupported`属性，默认该属性的取值为`false`，要启用异步处理支持，只需要将该属性设置为`true`即可。以`@WebServlet`为例，其配置方式如下所示：

   ```java
   @WebServlet(urlPatterns = "/async",asyncSupported = true)
   public class DomeAsyncServlet extends HttpServlet {...}
   ```

当开启Servlet异步支持后，就可以使用Servlet异步请求的特性了。要使用异步Servlet非常简单，只需要调用`HttpServletRequest`的`startAsync()`方法开启异步模式，并获取异步上下文`AsyncContext`：

```java
AsyncContext asyncContext = request.startAsync();
```

> `startAsync()`Javadoc描述如下：
>
> 将此请求放入异步模式，并使用原始(未包装的)ServletRequest和ServletResponse对象初始化其AsyncContext。

然后在调用`AsyncContext`的`start(Runnable run)`方法进行异步的业务处理，`start()`方法的入参是一个`Runnable`接口，它会新启一个线程以进行业务处理。需要注意的是，在`Runnable`对象中需要获取到`AsyncContext`引用，因为`AsyncContext`中保存有`HttpServletRequest`和`HttpServletResponse`的引用，业务逻辑部分需要使用`HttpServletRequest`获取请求数据以及使用`HttpServletResponse`以向客户端响应数据；当业务处理完毕之后，也需要`AsyncContext`告知Servlet容器业务逻辑处理完毕。

一个简单的异步处理的Servlet如下：

```java
@WebServlet(urlPatterns = "/dome/async",asyncSupported = true)
public class DomeAsyncServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/plain;charset=UTF-8");
        response.setCharacterEncoding("UTF-8");
        PrintWriter writer = response.getWriter();
        writer.write(String.format("进入Servlet时间：%s\n", LocalDateTime.now()));
        writer.flush();

        //开始异步上下文，并调用start异步处理
        AsyncContext ctx = request.startAsync();
        ctx.start(new Executor(ctx));

        writer.write(String.format("结束Servlet时间：%s\n", LocalDateTime.now()));
        writer.flush();
    }
}

public static class Executor implements Runnable{

    private final AsyncContext ctx;

    public Executor(AsyncContext ctx) {
        this.ctx = ctx;
    }

    @Override
    public void run() {
        try {
            //休眠十秒，模拟业务执行
            Thread.sleep(10000);
            PrintWriter writer = ctx.getResponse().getWriter();
            writer.write(String.format("业务处理完毕时间：%s\n", LocalDateTime.now()));
            writer.flush();
            //通知Servlet容器处理完毕
            ctx.complete();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

测试结果如下：

```
进入Servlet时间：2020-01-23T14:00:15.933182100
结束Servlet时间：2020-01-23T14:00:15.959114200
业务处理完毕时间：2020-01-23T14:00:25.959525600
```

测试成功，可以看到，测试结果确实如预期所料，*<结束Servlet时间>*先于*<业务处理完毕时间>*输出。

上面的代码看似OK，但实际上，通过`AsyncContext`的`start(Runnable)`方法进行异步处理是有问题的，将所有`writer.write(String.format("...时间：%s\n", LocalDateTime.now()));`代码修改如下：

```java
writer.write(String.format("...时间：%s，当前线程：%s\n", LocalDateTime.now(),Thread.currentThread().getName()));
```

再次测试输出结果如下：

```
进入Servlet时间：2020-01-23T14:27:45.373678500，当前线程：http-nio2-8080-exec-4
结束Servlet时间：2020-01-23T14:27:45.387642400，当前线程：http-nio2-8080-exec-4
业务处理完毕时间：2020-01-23T14:27:55.387877600，当前线程：http-nio2-8080-exec-5
```

从线程名可以看出，处理Servlet请求的线程和异步处理业务的线程的来源相同。际上，`AsyncContext`的`start(Runnable)`方法会从Servlet容器的线程池中获取新的线程，那么这样就有一个问题，先从池中获取一个线程用于处理Servlet请求，然后又从池中获取另一个线程异步处理业务，再将处理请求的线程还回到线程池中，这样看起来多此一举，并没有提高吞吐量，事实也确实如此。所以，实际在异步Servlet的请求中，一般都是自定义线程池去完成业务的处理。

一个自定义线程池处理Servlet请求示例如下：

```java
@WebServlet(urlPatterns = "/dome/async",asyncSupported = true)
public class DomeAsyncServlet extends HttpServlet {
    
    private ExecutorService executor = Executors.newFixedThreadPool(10);

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
		...

        //开始异步上下文，并调用start异步处理
        AsyncContext ctx = request.startAsync();
        executor.execute(new Executor(ctx));

		...
    }
}
```

测试结果如下：

```
进入Servlet时间：2020-01-23T14:26:14.397665700，当前线程：http-nio2-8080-exec-4
结束Servlet时间：2020-01-23T14:26:14.409634400，当前线程：http-nio2-8080-exec-4
业务处理完毕时间：2020-01-23T14:26:24.410332800，当前线程：pool-1-thread-1
```

### listener

除此之外，Servlet3.0还为异步处理提供了监听器，使用`AsyncListener`接口表示。它可以监控如下四种事件：

1. 异步线程开始时，调用AsyncListener的`onStartAsync(AsyncEvent event)`方法。
2. 异步线程超时时，调用AsyncListener的`onTimeout(AsyncEvent event)`方法。
3. 异步线程异常时，调用AsyncListener的`onError(AsyncEvent event)`方法。
4. 异步线程完成时，调用AsyncListener的`onComplete(AsyncEvent event)`方法。

要注册`AsyncListener`，只需要将准备好的`AsyncListener`对象传递给`AsyncContext`对象的`addListener(AsyncListener listener)`方法即可，如下所示：

```java
//为异步上下文设置监听器
ctx.addListener(new AsyncListener() {
        @Override
        public void onComplete(AsyncEvent event) throws IOException {

        }

        @Override
        public void onTimeout(AsyncEvent event) throws IOException {

        }

        @Override
        public void onError(AsyncEvent event) throws IOException {

        }

        @Override
        public void onStartAsync(AsyncEvent event) throws IOException {

        }
    });
```

注意:

1. 要触发`onTimeout(AsyncEvent event)`回调，需要设置超时时间，超时时间通过`AsyncContext`的`setTimeout(long timeout)`方法设置，单位毫秒，默认30000毫秒，即30秒。

   ```java
   ctx.setTimeout(5000);
   ```

2. `AsyncListener`必须在`request.startAsync();`调用之前添加，`onStartAsync(AsyncEvent event)`才会触发，但是在`request.startAsync();`调用之前，`AsyncContext`还不存在，所以在`request.startAsync();`第一次调用时，`onStartAsync(AsyncEvent event)`是不会被触发的，只有在超时后又重新开始的情况下才会起作用。

   一个简单的超时重试例子：

   ```java
   ctx.addListener(new AsyncListener() {
   			...
               @Override
               public void onTimeout(AsyncEvent event) throws IOException {
                   //超时，重试
                   event.getAsyncContext().dispatch();
               }
   			...
           });
   ```

   > 经过测试，只有通过`AsyncContext`的`dispatch`方法触发。

当首次请求进入Servlet时，`HttpServletRequest`对象的类型为`org.apache.catalina.connector.RequestFacade@xxxxxxxx`，当通过`AsyncContext`的`dispatch`重新请求时，`HttpServletRequest`对象的类型变为`org.apache.catalina.core.ApplicationHttpRequest@xxxxxxxx`。

在转发过程当中会携带上一次`AsyncContext`的中所有监听器，当本次请求再次重新执行`request.startAsync()`时，将会触发监听器`onStartAsync(AsyncEvent event)`函数的回调，并且会清理掉异步上下文中的所有监听器，如果不清理，在后面的代码中又会添加一模一样的监听器的新的实例，将会导致后续监听器中相同的代码多次执行。

### AsyncContext API

- `public ServletRequest getRequest();`

  获取当前异步请求的`HttpServletRequest`。

- `public ServletResponse getResponse();`

  获取当前异步请求的`HttpServletResponse`。

- `public boolean hasOriginalRequestAndResponse();`

  检查当前`AsyncContext`是否是使用原始request和response对象初始化的。

  在将请求置于异步模式之后，可在出站时根据此信息以判断在入站调用期间添加的请求*和/或*响应包装器在异步操作期间是否需要保留，或者是否需要释放。

  如果这个AsyncContext是通过调用`ServletRequest.startAsync()`用原始request和response对象初始化的，或者它是通过调用`ServletRequest.startAsync(ServletRequest, ServletResponse)`初始化的，并且ServletRequest和ServletResponse参数都没有携带任何应用程序提供的包装器，则为true；否则为false。

- `public void dispatch();`

  重新转发至当前Servlet。

- `public void dispatch(String path);`

- `public void dispatch(ServletContext context, String path);`

  重新转发至指定路径的Servlet。

- `public void start(Runnable run);`

  从Servlet容器的线程池中获取一个新的线程进行异步处理。

  > 不同的Servlet容器可能情况不同，具体情况需要具体分析，但是在Tomcat中测试是如此。

- `public void complete()`

  向Servlet容器报告异步请求成功。

- `public void addListener(AsyncListener listener);`

  `public void addListener(AsyncListener listener,ServletRequest servletRequest,ServletResponse servletResponse);`

  向异步上下文中添加异步监听器。

- `public <T extends AsyncListener> T createListener(Class<T> clazz)throws ServletException; `

  通过Class对象创建一个异步监听器。

- `public void setTimeout(long timeout);`

  设置异步请求的超时时间，单位毫秒，默认超时时间是30000毫秒。

- `public long getTimeout();`

  获取当前异步上下文的超时时间。

## 非阻塞IO

异步处理支持是Servlet 3.0提供的新特性，而非阻塞IO是Servlet 3.1提供的新特性，非阻塞IO是建立在异步处理支持之上的。

应用程序服务器中的Web容器通常为每个客户端请求使用一个服务器线程。要开发可伸缩的web应用程序，必须确保与客户端请求关联的线程永远不会处于空闲状态，等待阻塞操作完成。异步处理提供了一种机制来在新线程中执行特定于应用程序的阻塞操作，并立即将与请求关联的线程返回给容器。即在服务方法中对所有特定于应用程序的阻塞操作使用异步处理，由于input/output方面的考虑，与客户端请求相关的线程也可能暂时处于空闲状态。

如果客户端通过慢速网络链接提交一个大型HTTP POST请求，服务器读取请求的速度将快于客户端提供的速度。使用传统的I/O，与此请求关联的容器线程有时会处于空闲状态，等待请求的其余部分。

在异步模式下处理请求时，Java EE为servlet和过滤器提供了非阻塞I/O支持。以下步骤总结了如何使用非阻塞IO来处理请求并在服务方法中编写响应。

1. 将请求至于异步请求模式。
2. 获取`ServletInputStream`或`ServletOutputStream`输入输出流。
3. 将读监听器设置到`ServletInputStream`；将写监听器设置到`ServletOutputStream`。
4. 在监听器的回调方法中处理请求和响应

*非阻塞I/O支持：`javax.servlet.ServletInputStream`*

| Method                                            | Description                                       |
| ------------------------------------------------- | ------------------------------------------------- |
| `void setReadListener(ReadListener readListener)` | 设置读取监听器对象。                              |
| `boolean isReady()`                               | 如果可以不阻塞的情况下读取数据，则返回true。      |
| `boolean isFinished()`                            | 当从流中读取所有数据时，返回true，否则返回false。 |

*非阻塞I/O支持：`javax.servlet.ServletOutputStream`*

| Method                                               | Description                                    |
| ---------------------------------------------------- | ---------------------------------------------- |
| `void setWriteListener(WriteListener writeListener)` | 设置写监听器对象。                             |
| `boolean isReady()`                                  | 如果可以在不阻塞的情况下写入数据，则返回true。 |

*Listener接口非阻塞I/O支持：`javax.servlet.ReadListener`*

| Method                      | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| `void onDataAvailable()`    | 当ReadListener的实例被注册到ServletInputStream中时，容器将在第一次能够读取数据时调用该方法。随后，当且仅当调用了ServletInputStream.isReady()方法并返回false，并且随后可以读取数据时，容器才会调用该方法。 |
| `void onAllDataRead()`      | 读取当前请求的所有数据时调用。                               |
| `void onError(Throwable t)` | 处理请求时发生错误时调用。                                   |

*Listener接口非阻塞I/O支持：`javax.servlet.WriteListener`*

| Method                      | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| `void onWritePossible()`    | 当WriteListener的实例注册到ServletOutputStream时，容器将在第一次可以写入数据时调用此方法。随后，当且仅当调用了ServletOutputStream.isReady()方法并返回false，并且随后可以执行写操作时，容器才会调用该方法。 |
| `void onError(Throwable t)` | 在使用非阻塞api写入数据时发生错误时调用。                    |



一个简单的使用非阻塞IO读取大型的HTTP POST请求

```java
@Override
protected void doPost(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {
    response.setContentType("text/plain;charset=GBK");
    response.setCharacterEncoding("UTF-8");
    PrintWriter writer = response.getWriter();
    writer.write(String.format("start servlet time: %s, current thread: %s\n", LocalDateTime.now(),Thread.currentThread().getName()));
    writer.flush();
    
    //开始异步上下文，并调用start异步处理
    AsyncContext ctx = request.startAsync();

    ServletInputStream is =  request.getInputStream();
    is.setReadListener(new ReadListener() {
        byte buffer[] = new byte[1024];

        @Override
        public void onDataAvailable() throws IOException {
            System.out.println("ReadListener thread: " + Thread.currentThread().getName());

            int length = 0;
                boolean ready;
                do {
                    length += is.read(buffer);
                    ready = is.isReady();
                    System.out.println("ready = "+ready);
                } while (ready);
                System.out.println("读取了"+length+"字节的数据");
        }

        @Override
        public void onAllDataRead() throws IOException {
            System.out.println("数据读取完成");
            ctx.complete();
        }

        @Override
        public void onError(Throwable t) {
            System.out.println("读取数据异常");
        }
    });

    writer.write(String.format("stop  servlet time: %s, current thread: %s\n", LocalDateTime.now(),Thread.currentThread().getName()));
    writer.flush();
}
```

使用curl工具测试：

```shell
$ curl -X POST -F "file=@C:\Users\xlp\Desktop\XmanagerPowerSuite-6.0.0018_wm.exe" --limit-rate 5k http://localhost:8080/learn-servlet/dome/async
```

> - `-F`：指定上传的文件，格式为`"file=@<FILE_PATH>"`
>
> - `--limit-rate`：限制上传速率

客户端输出了：

```shell
start servlet time: 2020-01-26T15:22:02.733430600, current thread: http-nio2-8080-exec-4
stop  servlet time: 2020-01-26T15:22:02.746117300, current thread: http-nio2-8080-exec-4
```

控制台打印：

```
ReadListener thread: http-nio2-8080-exec-5
ready = true
ready = true
ready = true
ready = true
ready = true
ready = true
ready = true
ready = false
读取了8192字节的数据
ReadListener thread: http-nio2-8080-exec-6
ready = true
ready = true
ready = true
ready = true
ready = true
ready = true
ready = true
ready = false
读取了8192字节的数据
...
ready = false
读取了8192字节的数据
ReadListener thread: http-nio2-8080-exec-9
ready = true
ready = true
ready = true
ready = true
ready = true
ready = true
ready = true
ready = false
读取了8192字节的数据
ReadListener thread: http-nio2-8080-exec-6
ready = false
读取了90字节的数据
数据读取完成
```

每读取8次之后，就会切换为一个新的线程读取，并且每次切换的时读取的字节数都是8192个字节。根据结果可以看到，在每次读到第8次的时候，`is.isReady()`就返回`fasle`，表示不可读取，线程需要阻塞等待客户端的数据传输，但是因为是异步模式，所以会将当前线程返回到容器的线程池中去处理其他请求，待数据再次可读时，会再分配一个新的线程，并回调`onDataAvailable()`方法继续读取。

那么为什么都是读取8次，并且使8192个字节呢？

这是因为Servlet容器的默认输入缓冲区大小是`8 * 1024`个字节，而我们定义的读取缓冲区大小是1024个字节，所以需要8次才能读完，并且总大小为8192字节。但是因为服务端的读取速度快于客户端的数据传输速度，在每次读取完缓冲区中的8192字节的数据之后，在下一次缓冲区中的数据填充至少1024字节之前，是不可读的，这时线程将会因为IO而阻塞，所以`is.isReady()`返回`fasle`。

通过断点调试`ServletInputStream`对象的类型为`org.apache.catalina.connector.CoyoteInputStream`。

*关键源码如下：*

```java
public class CoyoteInputStream extends ServletInputStream {
    ...
    protected InputBuffer ib;

    protected CoyoteInputStream(InputBuffer ib) {
        this.ib = ib;
    }
    ...
}
```

在`CoyoteInputStream`对象中有一个`InputBuffer`成员变量，再进入到`InputBuffer`对象中，定义了`DEFAULT_BUFFER_SIZE`静态常量，即默认的缓冲区大小。

*关键源码如下：*

```java
public class InputBuffer extends Reader
    implements ByteChunk.ByteInputChannel, ApplicationBufferHandler {
    
    ......
    
    public static final int DEFAULT_BUFFER_SIZE = 8 * 1024;
    
    ......
    
    public InputBuffer() {
        this(DEFAULT_BUFFER_SIZE);
    }

    public InputBuffer(int size) {
        this.size = size;
        bb = ByteBuffer.allocate(size);
        clear(bb);
        cb = CharBuffer.allocate(size);
        clear(cb);
        readLimit = size;
    }
    ......
}
```

在`InputBuffer`中有两个构造方法，其中一个可以设置缓冲区的大小，但是实际上，并没有地方使用到它，也就是说没有可以设置缓冲区大小的入口。

> 因个人技术水平限制，这个结论不一定正确；如果有知道设置方式的，欢迎一起讨论。

通过断点调试，在`org.apache.catalina.connector.Request`类中使用到了`InputBuffer`对象：

```java
package org.apache.catalina.connector;

public class Request implements HttpServletRequest {
    ......
    protected final InputBuffer inputBuffer = new InputBuffer();

    protected CoyoteInputStream inputStream =
            new CoyoteInputStream(inputBuffer);

    ......
}
```

在`org.apache.catalina.connector.Request`类中，直接声明了`InputBuffer`的实例对象，并通过成员变量的形式和`CoyoteInputStream`的构造函数直接设置。

> 8192字节的情况是基于Tomcat容器，不能的容器可能情况不同。

每读取8192字节即被阻塞的情况是基于gradle的tomcat插件以及个人机器测试的情况，在不同的容器情形或不同的机器测试结果可能会不同。

例如，将项目war包直接放置在Tomcat 9的webapps目录下启动测试的结果如下：

```
ReadListener thread: http-nio-8080-exec-7
读取了524288字节的数据
ReadListener thread: http-nio-8080-exec-6
读取了851968字节的数据
......
读取了393216字节的数据
ReadListener thread: http-nio-8080-exec-5
读取了589824字节的数据
ReadListener thread: http-nio-8080-exec-7
读取了687992字节的数据
数据读取完成
```

这种情况下每次都是读取了数十万字节才会被阻塞一次，并且每次字节数都不固定。但不管怎样，它们的情形都是一样的。



看到这里可能会感到很疑惑：异步处理servlet和建立在异步处理支持之上的非阻塞I/O到底有什么关系？它们之间的功能看起来好像是重叠了，例如，完全可以使用异步Servlet进行I/O的读取，增加一个非阻塞IO好似多此一举？其实Servlet的异步IO只是对异步Servlet增强，它们两者之间没有必然的联系，虽然Servlet的异步IO是建立在异步Servlet之上。

异步Servlet的`AsyncContext`的`start(Runnable)`方法会使用Servlet线程池中的线程，所以一般是使用自定义线程池去执行异步操作。但是上面异步IO的测试结果可知，其也是使用的Servlet线程池中的线程，但是它是有意义的，因为在每次IO阻塞的时候，不要任何的线程等待，可以直接返回容器线程池中取处理其他业务，而异步Servlet即使被阻塞，也需要一个线程等待。



>  Servlet3的非阻塞是利用`java.util.EventListener`的事件驱动机制来实现的，它们实现了`EventListener`接口。
>
> ```java
> public interface ReadListener extends EventListener {...}
> public interface WriteListener extends EventListener {...}
> ```

## Appendix

### Resources

- [Servlet 3.0 新特性详解]: https://www.ibm.com/developerworks/cn/java/j-lo-servlet30/#major3

- [Servlet 4.0 入门]:https://www.ibm.com/developerworks/cn/java/j-javaee8-servlet4/index.html
- [Nonblocking I/O]: https://docs.oracle.com/javaee/7/tutorial/servlets013.htm
  

  
  