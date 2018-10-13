# Redis





```shell

$ wget http://download.redis.io/releases/redis-4.0.11.tar.gz
$ tar xzf redis-4.0.11.tar.gz
$ cd redis-4.0.11
$ make
```

```shell
make MALLOC=libc
```

requirepass 597646251

daemonize yes



```shell
yum install -y gcc
```







```
-rw-rw-r--.  1 root root 164219 8月   4 06:44 00-RELEASENOTES
-rw-rw-r--.  1 root root     53 8月   4 06:44 BUGS
-rw-rw-r--.  1 root root   1815 8月   4 06:44 CONTRIBUTING
-rw-rw-r--.  1 root root   1487 8月   4 06:44 COPYING
drwxrwxr-x.  6 root root    192 9月  29 16:17 deps
-rw-rw-r--.  1 root root     11 8月   4 06:44 INSTALL
-rw-rw-r--.  1 root root    151 8月   4 06:44 Makefile
-rw-rw-r--.  1 root root   4223 8月   4 06:44 MANIFESTO
-rw-rw-r--.  1 root root  20543 8月   4 06:44 README.md
-rw-rw-r--.  1 root root  58766 8月   4 06:44 redis.conf
-rwxrwxr-x.  1 root root    271 8月   4 06:44 runtest
-rwxrwxr-x.  1 root root    280 8月   4 06:44 runtest-cluster
-rwxrwxr-x.  1 root root    281 8月   4 06:44 runtest-sentinel
-rw-rw-r--.  1 root root   7921 8月   4 06:44 sentinel.conf
drwxrwxr-x.  3 root root   4096 9月  29 16:17 src
drwxrwxr-x. 10 root root    167 8月   4 06:44 tests
drwxrwxr-x.  8 root root   4096 8月   4 06:44 utils
```







centOS6.3 安装redis make报错 zmalloc.h:50:31: 错误：jemalloc/jemalloc.h：没有那个文件或目录
​	https://blog.csdn.net/maozherong/article/details/54236644
原因分析:

在redis的解压包下有个README文件，打开这个文件 有这个一段话。

llocator
---------


Selecting a non-default memory allocator when building Redis is done by setting
the `MALLOC` environment variable. Redis is compiled and linked against libc
malloc by default, with the exception of jemalloc being the default on Linux
systems. This default was picked because jemalloc has proven to have fewer
fragmentation problems than libc malloc.


To force compiling against libc malloc, use:


    % make MALLOC=libc


To compile against jemalloc on Mac OS X systems, use:


    % make MALLOC=jemalloc


Verbose build
-------------

说的是关于分配器allocator， 如果有MALLOC  这个 环境变量， 会有用这个环境变量的 去建立Redis。

而且libc 并不是默认的 分配器， 默认的是 jemalloc, 因为 jemalloc 被证明 有更少的 fragmentation problems 比libc。

但是如果你又没有jemalloc 而只有 libc 当然 make 出错。 所以加这么一个参数。

解决办法
make MALLOC=libc



## Redis集群教程

本文的是Redis集群的一般介绍，没有涉及复杂难懂的分布式概念的赘述，只是提供了从用户角度来如何搭建测试以及使用的方法，如果你打算使用并深入了解Redis集群，推荐阅读完本章节后，仔细阅读[Redis集群规范](http://www.redis.cn/topics/cluster-spec.html)一章。

本教程视图提供最终用户一个简单的关于集群和一致性特征的描述

请注意，本教程使用于Redis3.0（包括3.0）以上版本

如果你计划部署集群，那么我们建议你从阅读这个文档开始。

### Redis集群介绍

Redis集群是一个提供在多个Redis间节点间共享数据的程序集。

Redis集群并不支持处理多个keys的命令，因为这需要在不同的结点间移动数据，从而达不到像Redis那样的性能，在高负载的情况下可能会导致不可预料的错误。

Redis集群通过分区来提供**一定程度的可用性**，在实际环境中当某个结点宕机或者不可达的情况下继续处理命令。Redis集群的优势

- 自动分割数据到不同的节点上。
- 整个集群的部分结点失败或者不可达的情况下能够继续处理命令。

### Redis集群的数据分片

Redis集群没有使用一致性hash，而是引入了**哈希槽**的概念。

Redis集群有16384个哈希槽，每个key通过CRC16校验后对16384取模来决定放置那个槽。集群的每个节点负责一部分hash槽，举个例子，比如当前集群有3个结点，那么：

- 节点A包含0到5500号哈希槽。
- 节点B包含5501到11000号哈希槽。
- 节点C包含11001到16384号哈希槽。

这种结构很容易添加或者删除节点。比如如果我想新添加个结点D，我需要从节点A、B、C中得部分槽到D上。如果我想移除节点A，需要将A中的槽移到B和C节点上，然后将没有任何槽的A结点从集群中移除即可。由于从一个节点将哈希槽移到到另一个节点并不会停止服务，所以无论添加删除或者改变某个结点的哈希槽的数量都不会造成集群不可用的状态。

### Redis集群的主从复制模型

为了使在部分结点失败或者大部分结点都无法通信的情况下集群仍然可用，所以集群使用了主从复制模型，每个节点都会有N-1个复制品。

在我们的例子中具有A、B、C三个节点的集群，在没有复制模型的情况下，如果节点B失败了，那么整个集群就会以为缺少5501-1100这个范围的槽而不可用。

然而如果在集群创建的时候（或者过一段时间）我们为么个结点添加一个从节点A1、B1、C1，那么整个集群便有三个master节点和三个slave节点组成，这样在节点B失败后，集群便会选举B1为新的主节点继续服务，整个集群便不会因为槽找不到而不可用了。

不过大概B和B1都失败后，集群是不可用的。

### Redis一致性保证

Redis并不能保证数据的**强一致性**。这意味这在实际中集群在特定的条件下可能丢失写操作。

第一个原因是因为集群用了异步复制。写操作过程：

- 客户端向主节点B写入一条命令。
- 主节点B向客户端回复命令状态。
- 主节点将写操作复制给它的从节点B1、B2、B3。

主节点对命令的复制工作发生在返回命令回复之后，因为如果每次处理命令请求都需要等待复制操作完成的话，那么主节点处理命令请求的速度将极大地降低——我们必须在性能和一致性之间做出权衡。注意：Redis集群可能会在将来提供同步写的方法。Redis集群另外一种可能会丢失命令的情况是集群出现了网络分布，并且一个客户端与至少包括一个主节点在内的少数实例被孤立。

举个例子，假设集群包含A、B、C、A1、B1、C1六个节点，其中A、B、C为主节点，A1、B1、C1为A、B、C的从节点，还有一个客户端Z1，假设集群中发生网络分区，那么集群可能会分为两方，大部分的一方包含节点A、C、A1、B1和C1，小部分的一方则包含节点B和客户端Z1。

Z1仍然能够向主节点B中写入，如果网络分区发生时间较短，那么集群将会继续正常运作，如果分区的时间足够让大部分的一方将B1选举有新的master，那么Z1写入B中的数据便丢失了。

注意，在网络分裂出现期间，客户端Z1可以向主节点B发送写命令的最大时间是有限制的，这一时间限制称为节点超时时间（node timeout），是Redis集群的一个重要的配置选项。

## 搭建并使用Redis集群

搭建集群的第一件事情我们需要一些运行在集群模式的Redis实例，这意味着集群并不是有一些普通的Redis实例组成的，集群模式需要通过配置启用，开启集群模式后的Redis实例便可以使用集群特有的命令和特性了。

下面是一个最少选项的集群的配置文件:

```redis
port 6379
daemonize yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
appendonly yes
```

：用于开启实例的集群模式。

：设定了保存节点配置文件的路径，默认为`nodes.conf`。节点配置文件无须认为修改，它由Redis集群在启动时创建，并在有需要时进行更新。

要让集群正常运作至少需要三个主节点，不过在刚开始使用集群功能时，强烈建议使用六个节点：其中三个为主节点，而其余是哪个则是各个主节点的从节点。

首先，让我们进入一个新目录，并创建六个以端口号为名字的子目录，稍后我们将每个目录中运行一个Redis实例：命令如下：





### 搭建集群

现在我们已经有了六个正在运行中的Redis实例，接下来我们需要使用这些实例来创建集群，并为每个节点编写配置文件。通过使用Redis集群命令行工具`redis-trib`，编写节点配置文件的工作可以非常容易地完成：`redis-trib`位于Redis源码的src文件夹中，它是一个Ruby程序，这个程序通过向实例发送特殊命令来完成**创建新集群**，**检查集群**，或者**对集群进行重新分片（reshared）**等工作。

```shell
./redis-trib.rb create --replicas 1 192.168.56.2:6379 192.168.56.2:6380 192.168.56.4:6379 192.168.56.4:6380 192.168.56.5:6379 192.168.56.5:6380
```









```shell
/usr/bin/env: ruby: 没有那个文件或目录
```





安装ruby

```shell
[root@localhost ruby]# wget https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.1.tar.gz
[root@localhost ruby]# tar -zxvf ruby-2.5.1.tar.gz
[root@localhost ruby]# cd ruby-2.5.1
[root@localhost ruby]# ./configure
[root@localhost ruby]# make
[root@localhost ruby]# make install
[root@localhost ruby]# ruby -v
ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-linux]
```

安装RubyGems

```shell
yum install -y rubygems
```



```shell
wget https://rubygems.org/rubygems/rubygems-2.7.7.tgz
tar -zxvf rubygems-2.7.7.tgz
cd rubygems-2.7.7


cannot load such file -- zlib (LoadError)


yum install -y zlib-devel

cd ruby/ruby-2.5.1/ext/zlib

ruby ./extconf.rb

[root@localhost zlib]# make
make: *** 没有规则可以创建“zlib.o”需要的目标“/include/ruby.h”。 停止。

zlib.o: $(top_srcdir)/include/ruby.h

zlib.o: ../../include/ruby.h

[root@localhost zlib]# make
compiling zlib.c
linking shared-object zlib.so
[root@localhost zlib]# make install
/usr/bin/install -c -m 0755 zlib.so /usr/local/lib/ruby/site_ruby/2.5.0/x86_64-linux


[root@localhost rubygems-2.7.7]# gem install redis
ERROR:  While executing gem ... (Gem::Exception)
    Unable to require openssl, install OpenSSL and rebuild Ruby (preferred) or use non-HTTPS sources
    
yum install openssl-devel
cd ruby/ruby-2.5.1/ext/openssl
 ruby extconf.rb 
[root@localhost openssl]# make
compiling openssl_missing.c
make: *** 没有规则可以创建“ossl.o”需要的目标“/include/ruby.h”。 停止。

make && make install
```





```shell
[root@localhost openssl]# gem install redis
Fetching: redis-4.0.2.gem (100%)
Successfully installed redis-4.0.2
Parsing documentation for redis-4.0.2
Installing ri documentation for redis-4.0.2
Done installing documentation for redis after 1 seconds
1 gem installed
```





```shell
[ERR] Sorry, can't connect to node 192.168.56.2:6380
```

bind

protected

requirepass



# [redis创建集群——[ERR\] Sorry, can't connect to node 192.168.X.X](https://www.cnblogs.com/lmy2018/p/8514787.html)



**redis默认只允许本地访问，要使redis可以远程访问可以修改redis.conf**

 

**解决办法：注释掉bind 127.0.0.1可以使所有的ip访问redis**

 

若是想指定多个ip访问，但并不是全部的ip访问，可以bind

 

 

在redis3.2之后，redis增加了protected-mode，在这个模式下，即使注释掉了bind 127.0.0.1，再访问redisd时候还是报错

**修改办法：protected-mode no**





https://www.cnblogs.com/lmy2018/p/8514787.html

https://blog.csdn.net/xiaobo060/article/details/80616718

https://blog.csdn.net/feinifi/article/details/78251486

https://blog.csdn.net/zhengwei125/article/details/80019887

https://blog.csdn.net/weijifeng_/article/details/80115093



执行成功之后

```shell
[root@localhost src]# ./redis-trib.rb create --replicas 1 192.168.56.2:6379 192.168.56.2:6380 192.168.56.4:6379 192.168.56.4:6380 192.168.56.5:6379 192.168.56.5:6380
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.56.2:6379
192.168.56.4:6379
192.168.56.5:6379
Adding replica 192.168.56.4:6380 to 192.168.56.2:6379
Adding replica 192.168.56.5:6380 to 192.168.56.4:6379
Adding replica 192.168.56.2:6380 to 192.168.56.5:6379
M: 73abb17d55a3087a036f2833f25ac09ce66db55d 192.168.56.2:6379
   slots:0-5460 (5461 slots) master
S: 42f5ec2af130650c0ef08680c3c95b01d420fb76 192.168.56.2:6380
   replicates 6330d9a4e55823149636a318d0588d5fc04be67c
M: d5b7a3d14e26ffd6f5deed9f769d26ba5c1bd335 192.168.56.4:6379
   slots:5461-10922 (5462 slots) master
S: ef7e3d917b0cd450efec5b43265b78adacacc37b 192.168.56.4:6380
   replicates 73abb17d55a3087a036f2833f25ac09ce66db55d
M: 6330d9a4e55823149636a318d0588d5fc04be67c 192.168.56.5:6379
   slots:10923-16383 (5461 slots) master
S: 9287acc5d6a96c9b1762a53f50fd0ead09ded1c9 192.168.56.5:6380
   replicates d5b7a3d14e26ffd6f5deed9f769d26ba5c1bd335
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.......
>>> Performing Cluster Check (using node 192.168.56.2:6379)
M: 73abb17d55a3087a036f2833f25ac09ce66db55d 192.168.56.2:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 9287acc5d6a96c9b1762a53f50fd0ead09ded1c9 192.168.56.5:6380
   slots: (0 slots) slave
   replicates d5b7a3d14e26ffd6f5deed9f769d26ba5c1bd335
S: 42f5ec2af130650c0ef08680c3c95b01d420fb76 192.168.56.2:6380
   slots: (0 slots) slave
   replicates 6330d9a4e55823149636a318d0588d5fc04be67c
M: d5b7a3d14e26ffd6f5deed9f769d26ba5c1bd335 192.168.56.4:6379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: 6330d9a4e55823149636a318d0588d5fc04be67c 192.168.56.5:6379
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: ef7e3d917b0cd450efec5b43265b78adacacc37b 192.168.56.4:6380
   slots: (0 slots) slave
   replicates 73abb17d55a3087a036f2833f25ac09ce66db55d
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



使用`redis-cli`登入Redis客户端，会有如下错误提示

```shell
[root@localhost src]# ./redis-cli
127.0.0.1:6379> setex name 10 redis
(error) MOVED 5798 192.168.56.4:6379
```



设置一个值，它会告诉你“重定向到位置为192.168.56.4:6379的槽[5798]”

```shell
[root@localhost src]# ./redis-cli -c
127.0.0.1:6379> setex name 10 redis
-> Redirected to slot [5798] located at 192.168.56.4:6379
OK
```



查看集群信息

查看集群信息使用命令`cluster info`:

```shell
192.168.56.4:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:3
cluster_stats_messages_ping_sent:685
cluster_stats_messages_pong_sent:678
cluster_stats_messages_meet_sent:5
cluster_stats_messages_sent:1368
cluster_stats_messages_ping_received:678
cluster_stats_messages_pong_received:690
cluster_stats_messages_received:1368
```

