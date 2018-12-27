## Redis Pipeline

Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。

这意味着通常情况下一个请求会遵循以下步骤：

- 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
- 服务端处理命令，并将结果返回给客户端。

因此，例如下面是4个命令序列执行情况：

- *Client*:INCR X
- *Server*:1
- *Client*:INCR X
- *Server*:2
- *Client*:INCR X
- *Server*:3
- *Client*:INCR X
- *Server*:4

客户端和服务器通过网络进行连接。这个连接可以很快（loopback接口）或很慢（建立了一个多次跳转的网络链接）。无论网络延迟如何延时，数据包总是能从客户端到到达服务器，并从服务器返回数据回复客户端。

这个时间被称之为RTT（Round Trip Time - 往返时间）。当客户端需要在一个批处理中执行多次请求时很容易看到这是如何影响性能的（例如添加许多原道同一个list，或者用很多Keys填充数据库）。例如，如果RTT时间是250毫秒（在一个很慢的连接下），即使服务器每秒能处理100k的请求数，我们每秒最多只能处理4个请求。

如果采用loopback接口，RTT就短得多（比如我的主机ping 127.0.0.1只需要44毫秒），但它在一次批量写入操作中任然是一笔很大的开销。

幸运的是有一种方法可以改善这种情况。

### Redis管道（Pipelining）

一次请求/响应服务器能实现处理新的请求，即使旧的请求还未被响应。这样就可以将多个命令发送到服务器，而不用等待回复，最后在一个步骤中服务该答复。

这就是管道（pipelining），是一种几十年来广泛使用的技术。例如许多P0P3协议已经实现支持这个功能，大大加快了从服务器下载新邮件的过程。

Redis很早就支持管道（pipelining）技术，因此无论你运行的是什么版本，你都可以使用观管道（pipelining）操作Redis。下面是一个使用的例子：

```shell
$ (printf "PING\r\nPING\r\nPING\r\n"; sleep 1) | nc localhost 6379
+PONG
```

这一次我们没有为每个命令都花费RTT开销，而是只用了一个命令的开销时间。

非常明确的，用管道顺序操作的第一个例子如下：

- *Client*:INCR X
- *Client*:INCR X
- *Client*:INCR X
- *Client*:INCR X
- *Server*:1
- *Server*:2
- *Server*:3
- *Server*:4

**重要说明：**使用管道发送命令时，服务器将被迫回复一个队列答复，占用很多内存。所以，如果你需要发送大量的命令，最好是把他们按照合理数量分批次的处理，例如10K的命令，读回复，然后再发送另一个10k的命令，等等。这样速度几乎是相同的，但是在回复这10k命令队列需要非常大量的内存用来组织返回数据内容。

### 一些测试

下面我们会使用Spring Data Redis客户端进行一些使用管道和不使用管道的情况，测试管道技术对速度的提升效果：

```java
@Component
public class Pipeline implements ApplicationRunner {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    private static final String[] ids = new String[]{
            "38fcda29-1307-4148-932c-8ef42c13df7d",
        	"f89c015f-d41f-4840-b6ec-0b0a5c0b4ef5",
        	"88ba6214-b768-4ddb-8faf-185eed718444",
        	"a208b422-8ac3-48e0-9196-2ccd49a09d3b",
        	"9dfd9633-5b34-4244-904b-b5a07e864ae9",
        	...};//100个

    private StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
    private GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = 
        new GenericJackson2JsonRedisSerializer();

    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        useExecutePipelined();
        notUsePipelined();
        usePipelined();
    }

    
   /**
     * 使用 {@link RedisTemplate#execute(RedisCallback)} 执行pipeline.
     */
    public void useExecutePipelined(){
        long startTime = System.currentTimeMillis();
        stringRedisTemplate.execute(connection -> {
            connection.openPipeline();
            for (String id : ids) {
                connection.get(stringRedisSerializer.serialize(id));
            }
            List<Object> objs = connection.closePipeline();
            return null;
        });
        System.out.println("use execute pipelined : "+
                           (System.currentTimeMillis() - startTime)+" ms");
    }

    
    
    /**
     * 不使用pipeline。
     */
    public void notUsePipelined(){
        long startTime = System.currentTimeMillis();
        ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();
        for (String id : ids) {
            String user = operations.get(id);
        }
        System.out.println("not use pipelined : "+
                           (System.currentTimeMillis() - startTime)+" ms");
    }

    
    
    /**
     * 使用 {@link RedisTemplate#executePipelined(RedisCallback)} 执行pipeline.
     */
    public void usePipelined(){
        long startTime = System.currentTimeMillis();
        stringRedisTemplate.executePipelined((RedisCallback<?>) connection -> {
            for (String id : ids) 
                connection.get(stringRedisSerializer.serialize(id));
            return null;
        },genericJackson2JsonRedisSerializer);
        System.out.println("use pipelined : "+
                           (System.currentTimeMillis() - startTime)+" ms");
    }
}
```

写了三个测试方法，分别是`useExecutePipelined()`、`notUsePipelined()`、`usePipelined()`，测试输出结果如下：

```
use execute pipelined : 1138 ms
not use pipelined : 119 ms
use pipelined : 17 ms
```

看到这个结果可能会感觉很奇怪，第一次我们使用pipeline，第二次没有使用，第三次使用了，但是第一次使用pipeline的执行时长比第二次没有使用pipeline的执行时长还长，不是说使用pipeline更快么，为什么结果跟预期的不一样，关于这个问题只需要一句话解释：**首次初始化建立连接**。

如果将run()方法改成如下：

```java
@Override
public void run(ApplicationArguments args) throws Exception {
    useExecutePipelined();
    useExecutePipelined();
    notUsePipelined();
    usePipelined();
}
```

`useExecutePipelined()`方法执行两次，输出结果：

```
use execute pipelined : 1172 ms
use execute pipelined : 19 ms
not use pipelined : 122 ms
use pipelined : 10 ms
```



第一种方式由我们自己控制pipeline的打开与关闭，实际上RedisTempleate以及封装了pipeline的打开与关闭，我们只需要设置其参数进项控制就可以了。`RedisTemplate#execute(RedisCallback, boolean, boolean)`第三个参数控制是否开启pipeline，具体情况请查看源码。

示例如下：

```java
/**
  * @see RedisTemplate#execute(RedisCallback, boolean, boolean)
  */
public void useExecutePipelined1(){
    Object execute = stringRedisTemplate.execute(connection -> {
        for (String id : ids) {
            connection.get(stringRedisSerializer.serialize(id));
        }
        return null;
    },false,true);
    System.out.println("execute pipelined result : "+execute);
}
```

输出结果：

```
execute pipelined result : null
```

使用这种方式在外部不能直接拿到返回结果，适用于添加数据。

根据上面的结果数据表明，开启了管道操作后，往返延时以及被改善的相当低了。

```
use execute pipelined : 19 ms
not use pipelined : 122 ms
use pipelined : 10 ms
```

如你所见，开启管道后，我们的速度效率提升了5-10呗。

### 管道（Pipelining）VS 脚背（Scripting）

大量pipeline应用场景可通过Redis脚本（Redis版本>=2.6）得到更搞笑的处理，后者在服务器端执行大量工作。脚本的一大优势是可通过最小的延迟读写数据，让读、计算、写等操作变得非常快（pipeline在这种情况下不能使用，因为客户端在写命令前需要读命令返回的结果）。

应用程序有时可能在pipeline中发送`eval`或`evalsha`命令。Redis通过`script load`命令（保证`evalsha`成功被调用）明确支持这种情况。