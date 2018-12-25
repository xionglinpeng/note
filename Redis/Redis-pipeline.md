## Redis Pipeline









### Code Operation

下面是使用`Spring-Data-Redis`API编写的代码示例：

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