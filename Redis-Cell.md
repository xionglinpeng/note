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

## Usage

从Redis（尝试运行`redis-cli`）使用模块加载的新`CL.THROTTLE`命令。它是这样使用的：

```shell
CL.THROTTLE <key> <max_burst> <count per period> <period> [<quantity>]
```

- `key`：`key`是对其进行速率限制的标识符。可能的例子是:
  - 用户帐户的唯一标识符。
  - 传入请求的原始IP地址。
  - 一个静态字符串(例如全局字符串)，用于限制整个系统的操作。

- `max_burst`：

- `count per period`：

- `period`：

- `quantity`：

For Example:

```shell
CL.THROTTLE user123 15 30 60 1
               ▲     ▲  ▲  ▲ ▲
               |     |  |  | └───── apply 1 token (default if omitted)
               |     |  └──┴─────── 30 tokens / 60 seconds
               |     └───────────── 15 max_burst
               └─────────────────── key "user123"
```

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

The meaning of each array item is:

1. Whether the action was limited:
   - `0` indicates the action is allowed.
   - `1` indicates that the action was limited/blocked.
2. The total limit of the key (`max_burst` + 1). This is equivalent to the common `X-RateLimit-Limit` HTTP header.
3. The remaining limit of the key. Equivalent to `X-RateLimit-Remaining`.
4. The number of seconds until the user should retry, and always `-1` if the action was allowed. Equivalent to `Retry-After`.
5. The number of seconds until the limit will reset to its maximum capacity. Equivalent to `X-RateLimit-Reset`.

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



127.0.0.1:6379> CL.THROTTLE
(error) ERR unknown command `CL.THROTTLE`, with args beginning with:

127.0.0.1:6379> CL.THROTTLE
(error) Cell error: Usage: cl.throttle <key> <max_burst> <count per period> <period> [<quantity>]





jave的jar包下载：http://www.java2s.com/Code/Jar/j/Downloadjave102jar.htm

jave依赖ffmpeg：https://ffmpeg.org/download.html

```shell
mvn install:install-file -Dfile=jave-1.0.2.jar -DgroupId=it.sauronsoftware.jave -DartifactId=jave -Dversion=1.0.2
```













添加jar包到本地Maven仓库
​          在使用Maven的过程中，经常碰到有些jar包在中央仓库没有的情况。如果公司有私服，那么就把jar包安装到私服上。如果没有私服，那就把jar包安装到本地Maven仓库。今天介绍2种安装jar包到本地Maven仓库的方法，下面进入正题。
一、使用Maven命令安装jar包
​        前提：在windows操作系统中配置好了Maven的环境变量，怎么配置请自己百度，这里不介绍，可参考https://jingyan.baidu.com/article/cb5d61050b8ee7005d2fe04e.html

在windows的cmd命令下，参考下面安装命令安装jar包。注意：这个命令不能换行，中间用空格来分割的

 ```shell
安装指定文件到本地仓库命令：mvn install:install-file
-DgroupId=<groupId>       : 设置项目代码的包名(一般用组织名)
-DartifactId=<artifactId> : 设置项目名或模块名 
-Dversion=1.0.0           : 版本号
-Dpackaging=jar           : 什么类型的文件(jar包)
-Dfile=<myfile.jar>       : 指定jar文件路径与文件名(同目录只需文件名)
安装命令实例：
mvn install:install-file -DgroupId=com.baidu -DartifactId=ueditor -Dversion=1.0.0 -Dpackaging=jar -Dfile=ueditor-1.1.2.jar
 ```




