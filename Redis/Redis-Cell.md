# redis-cell

一个Redis模块，它在Redis中作为单个命令提供速率限制。实现了相当复杂的通用单元格速率算法(GCRA)，它提供了滚动时间窗口，不依赖于后台滴水过程。

Redis公开的原语非常适合处理速率限制，但是由于它不是内置的，公司和组织在Redis的基础上使用基本命令和Lua脚本实现自己的速率限制逻辑是很常见的（举个例子，我在Heroku和Stripe公司见过这个例子）。这通常会导致不成熟的实现，需要进行一些尝试才能正确。指令redis-cell提供了一种与语言无关率限制器，可以轻松地插入到许多云架构中。

非正式的基准测试表明，redis-cell非常快，跑起来比基本的Redis要慢两倍（从Redis客户端看，每个命令大约0.1毫秒）。

## Install

[用于Mac、Linux Frssbsd和Linux Gnu操作系统的redis-cell二进制文件](https://github.com/brandur/redis-cell/releases)。如果有兴趣为当前不支持的体系结构或操作系统开发二进制文件，就打开一个issue 。

下载并提取library，然后将其移动到Redis可以访问的地方（注意：Mac稳定版本的扩展名将是`.dylib`，而不是`.so`）。

```shell
$ wget https://github.com/brandur/redis-cell/releases/download/v0.2.2/redis-cell-*.tar.gz
$ tar -zxvf redis-cell-*.tar.gz
$ cp libredis_cell.so /path/to/modules/
```

> Assert:
>
> - [**redis-cell-v0.2.2-x86_64-apple-darwin.tar.gz**](https://github.com/brandur/redis-cell/releases/download/v0.2.2/redis-cell-v0.2.2-x86_64-apple-darwin.tar.gz)
> - [**redis-cell-v0.2.2-x86_64-unknown-freebsd.tar.gz**](https://github.com/brandur/redis-cell/releases/download/v0.2.2/redis-cell-v0.2.2-x86_64-unknown-freebsd.tar.gz)
> - [**redis-cell-v0.2.2-x86_64-unknown-linux-gnu.tar.gz**](https://github.com/brandur/redis-cell/releases/download/v0.2.2/redis-cell-v0.2.2-x86_64-unknown-linux-gnu.tar.gz)
>
> 注意：上面的版本对应的系统环境分别为`apple-darwin`、`freebsd`、`linux-gnu`，请根据对应的环境下载响应的二进制包。

或者，从源代码克隆并构建项目。要做到这一点，您需要安装Rust(如果您是在Mac上，这可能与brew install Rust一样简单)。

```shell
$ git clone https://github.com/brandur/redis-cell.git
$ cd redis-cell
$ cargo build --release
$ cp target/release/libredis_cell.dylib /path/to/modules/
```

**Note: that Rust 1.13.0+ is required.**

运行Redis加载新的模块

```shell
$ redis-server --loadmodule /path/to/modules/libredis_cell.so
```

或者将一下内容添加到Redis配置文件中

```shell
################################## MODULES #####################################
# 启动时加载模块。如果服务器不能加载模块，它将中止。可以使用多个loadmodule指令。
#
loadmodule /path/to/my_module.so
loadmodule /path/to/other_module.so
```

**测试**

安装前执行`CL.THROTTLE`：

```shell
127.0.0.1:6379> CL.THROTTLE
(error) ERR unknown command `CL.THROTTLE`, with args beginning with:
```

安装后执行`CL.THROTTLE`：

```shell
127.0.0.1:6379> CL.THROTTLE
(error) Cell error: Usage: cl.throttle <key> <max_burst> <count per period> <period> [<quantity>]
```

## Usage

从Redis（尝试运行`redis-cli`）使用模块加载的新`CL.THROTTLE`命令。它是这样使用的：

```shell
CL.THROTTLE <key> <max_burst> <count per period> <period> [<quantity>]
```

- `key`：`key`是对其进行速率限制的标识符。可能的例子是:
  - 用户帐户的唯一标识符。
  - 传入请求的原始IP地址。
  - 一个静态字符串(例如全局字符串)，用于限制整个系统的操作。

- `max_burst`：桶的容量

- `count per period`：每period时间消耗的容量

- `period`：时间

- `quantity`：每次消耗的容量

For Example:

```shell
CL.THROTTLE user123 15 30 60 1
               ▲     ▲  ▲  ▲ ▲
               |     |  |  | └───── apply 1 token (default if omitted)
               |     |  └──┴─────── 30 tokens / 60 seconds
               |     └───────────── 15 max_burst
               └─────────────────── key "user123"
```

**理解（重要）：**

为了更好的说明这几个参数的意义，所以这里再使用文字对上面的例子进行描述：

首先`user123`指的是这个令牌桶限流的键，就跟string或其他类型的键一样，没有任何区别。

15指的是令牌桶的容量，即这个容量满了之后开始限流

30指的是在指定的时间内，会被消耗的容量。

60是一个时间，单位秒，即指的是60秒内会消耗多少容量，例如上面的30 60，即是说明60秒内会消耗30的容量，30/60的值即是每秒会消耗的容量，30/60=1/2，即每秒消耗二分之一的容，每两秒消耗一的容量。也就说桶的容量装满了之后，开始限流，但是每两秒消耗一的容量，消耗之后空出了空间才能添加，即是说每两秒才能添加1容量。换算成http请求，开始限流之后，每两秒才会接收一次请求。以此类推，`1 3`,`2 8`,`100 500`也是一样的。

1表示每次向桶中添加的容量，但是如果这个值是2，3，或者其他大于1的值呢？无论这个值是几，桶任然还是按1容量为单位进行消耗，例如`2 10 2`，每5秒消耗1容量，但是令牌桶每次需要两个容量才能添加，所以也就需要消耗了2个容量才能添加，需要10秒，换算成http请求，开始限流之后，10秒才会接收一次请求。依次类推，计算限流之后多少秒之后才会接受一次请求，只需要将2变量1，对应的10相应较少，然后再乘以2即可。

## Resource

这意味着一个令牌(最后一个参数中的1)应该应用于键user123的速率限制。

密钥上的30个令牌允许超过60秒，最大初始爆发为15个令牌。

每次调用都会提供速率限制参数，以便可以在运行时轻松地重新配置限制。

该命令将响应一个整数数组:

```
127.0.0.1:6379> CL.THROTTLE user123 15 30 60
1) (integer) 0   # 0 表示允许，1表示拒绝
2) (integer) 15  # 漏斗容量capacity
3) (integer) 14  # 漏斗剩余空间left_quota
4) (integer) -1  # 如果拒绝了，需要多长时间后再试(漏斗有空间了，单位秒)
5) (integer) 2   # 多长时间后，漏斗完全空出来(left_quota==capacity，单位秒)...
```

每个数组项的含义是:

这一个是原因文档，不能理解

The meaning of each array item is:

1. Whether the action was limited:
   - `0` indicates the action is allowed.
   - `1` indicates that the action was limited/blocked.
2. The total limit of the key (`max_burst` + 1). This is equivalent to the common `X-RateLimit-Limit` HTTP header.
3. The remaining limit of the key. Equivalent to `X-RateLimit-Remaining`.
4. The number of seconds until the user should retry, and always `-1` if the action was allowed. Equivalent to `Retry-After`.
5. The number of seconds until the limit will reset to its maximum capacity. Equivalent to `X-RateLimit-Reset`.

按照漏斗的原理理解：

每个数组项的含义是：

1. 是否允往漏斗中添加指定单位的水

   - 0：允许

   - 1：拒绝

2. 漏斗总容量。注意，容量计数是从0开始的，所以实际漏斗容量需要+1。

3. 漏斗剩余容量.。

4. 重试时间，单位：秒。

   1. 如果这个值为-1，表示可以往漏斗中添加指定单位的水；
   2. 如果不为-1，那么说明漏斗已经满了，并且这个值表示需要多长时间漏斗才会漏出指定单位的水。

5. 表示如果当前停止添加水，多长时间漏斗中的水才会漏完。单位：秒。

## Multiple Rate Limits

通过使用不同的键名实现不同类型的速率限制:

```shell
CL.THROTTLE user123-read-rate 15 30 60
CL.THROTTLE user123-write-rate 5 10 60
```

## On Rust

redis-cell是用Rust编写的，它使用该语言的FFI模块与Redis自己的模块系统交互。Rust非常适合于此，因为它不需要GC，并且是通过一个很小的运行时引导的。

这个库的作者认为用Rust而不是C来编写模块可以传达类似的性能特征，但是这样的实现更有可能避免很多C程序中常见的错误和内存缺陷。

## License

这是麻省理工学院许可条款下的自由软件(详见文件许可)。

## Githup

https://github.com/brandur/redis-cell

## Client Used

Redis-Cell模块提供的命令并没有在客户端库进行封装，包括`Jedis`和`Lettuce`等，那么在客户端需要如何调用呢？可以使用Lua脚本调用。

Redis-Cli客户端Lua脚本调用：

```shell
eval "return redis.call('CL.THROTTLE',KEYS[1],ARGV[1],ARGV[2],ARGV[3],ARGV[4])" 1 user123 15 30 60 1
```

RedisTemplate封装Lua脚本调用（封装为Spring Boot自动装配）：

```java
import lombok.Data;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.RedisScript;

import java.util.Collections;
import java.util.List;
import java.util.Objects;

public class ThrottleService {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Autowired
    private RedisScript<List<Long>> throttlescript;

    private static final ThrottleMessage message = new ThrottleMessage();

    public ThrottleService(RedisScript<List<Long>> throttlescript) {
        this.throttlescript = throttlescript;
    }

    /**
     * 漏斗限流
     * 每次加水量默认为1单位
     * @param throttleKey 限流标识
     * @param capacity 漏斗容量
     * @param expend 单位时间漏水量
     * @param second 单位时间
     * @return 限流信息
     */
    public ThrottleMessage throttle(String throttleKey, String capacity, String expend, String second) {
        return throttle(throttleKey, capacity, expend, second,"1");
    }

    /**
     * 漏斗限流
     * expend/second = (expend/expend)/(second/expend) = 1/N => 每N秒漏1单位水
     * 当漏斗满时，需要N*units秒才可以加水
     * @param throttleKey 限流标识
     * @param capacity 漏斗容量
     * @param expend 单位时间漏水量
     * @param second 单位时间
     * @param units 每次加水量
     * @return 限流信息
     */
    public ThrottleMessage throttle(String throttleKey, String capacity, String expend, String second, String units) {
        List<Long> data = stringRedisTemplate.execute(throttlescript,
                Collections.singletonList(throttleKey),capacity,expend,second,units);
        ThrottleMessage throttleMessage = message.clone();
        if (Objects.nonNull(data)) {
            throttleMessage.setIsAllow(data.get(0) == 0);
            throttleMessage.setCapacity(data.get(1));
            throttleMessage.setResidue((data.get(2)));
            throttleMessage.setRetry(data.get(3));
            throttleMessage.setLeakage(data.get(4));
        }
        return throttleMessage;
    }

    @Data
    public static class ThrottleMessage implements Cloneable {

        /**
         * 是否允往漏斗中添加指定单位的水
         *  0：允许
         *  1：拒绝
         */
        private Boolean isAllow;

        /**
         * 漏斗总容量。
         */
        private Long capacity;

        /**
         * 漏斗剩余容量.。
         */
        private Long residue;

        /**
         * 重试时间，单位：秒。
         *  如果这个值为-1，表示可以往漏斗中添加指定单位的水；
         *  如果不为-1，那么说明漏斗已经满了，并且这个值表示需要多长时间漏斗才会漏出指定单位的水。
         */
        private Long retry;

        /**
         * 表示如果当前停止添加水，多长时间漏斗中的水才会漏完。单位：秒。
         */
        private Long leakage;

        @Override
        public ThrottleMessage clone(){
            try {
                return (ThrottleMessage)super.clone();
            } catch (CloneNotSupportedException e) {
                e.printStackTrace();
            }
            return this;
        }
    }

}
```

自动装配

```java
package com.threes.redis.script;

import org.springframework.boot.autoconfigure.AutoConfigureAfter;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.RedisScript;
import org.springframework.scripting.ScriptSource;
import org.springframework.scripting.support.ResourceScriptSource;

import java.io.File;
import java.io.IOException;
import java.util.List;

@AutoConfigureAfter(RedisAutoConfiguration.class)
@ConditionalOnBean(StringRedisTemplate.class)
public class RedisScriptAutoConfiguration {

    private static final String SCRIPT_PATH = "META-INF"+ File.separator +"scripts"+ File.separator;

    @Bean
    @ConditionalOnClass(RedisScript.class)
    @ConditionalOnMissingBean
    public RedisScript throttleRedisScript() throws IOException {
        ScriptSource scriptSource = new ResourceScriptSource(
                new ClassPathResource(SCRIPT_PATH +"throttle.lua"));
        return RedisScript.of(scriptSource.getScriptAsString(), List.class);
    }

    @Bean
    @ConditionalOnMissingBean
    public ThrottleService throttleService(RedisScript<List<Long>> redisScript){
        return new ThrottleService(redisScript);
    }

}
```

内部类方式：

```java
List execute = redisTemplate.execute(new RedisScript<List>() {
    @Override
    @NonNull
    public String getSha1() {
        return "90483b568570cfb5f52d8d278e2bc1c1e11f9c64";
    }

    @Override
    public Class<List> getResultType() {
        return List.class;
    }

    @Override
    @NonNull
    public String getScriptAsString() {
        return "return redis.call('CL.THROTTLE',KEYS[1],ARGV[1],ARGV[2],ARGV[3],ARGV[4])";
    }
}, Collections.singletonList("user123"), "15", "30", "60", "1");
System.out.println(execute);//[0, 16, 15, -1, 2]
```

