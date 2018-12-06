# Redis



## redis安装

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

make[3]: gcc：命令未找到

```shell
yum install -y gcc
```

zmalloc.h:50:31: 致命错误：jemalloc/jemalloc.h：没有那个文件或目录

make MALLOC=libc

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



为了方便操作，配置redis的环境变量

```shell
vim /etc/profile

#redis
export REDIS_SRC=/usr/redis/redis-4.0.11/src
PATH=$PATH:${REDIS_SRC}

source /etc/profile

[root@localhost redis-4.0.11]# redis-cli
Could not connect to Redis at 127.0.0.1:6379: Connection refused
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected>
```











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








### 原生命令安装

这种方式只是在于学习理解时使用，不建议在实际应用环境中使用。

#### 准备节点

首先创建一个目录config，用于放置redis配置文件，创建一个data目录，用于放置redis集群数据日志及配置文件。

```shell
[root@localhost ~]# cd /usr/redis/
[root@localhost redis]# mkdir redis-cluster
[root@localhost redis]# cd redis-cluster/
[root@localhost redis-cluster]# mkdir config
[root@localhost redis-cluster]# mkdir data
```

创建redis配置文件

```shell
[root@localhost redis-cluster]# vim redis-6379.config
```

配置文件的内容如下：

```shell
################################## NETWORK #####################################
port 6379
################################# GENERAL #####################################
daemonize yes
logfile "6379.log"
################################ SNAPSHOTTING  ################################
dbfilename "dump-6379.rdb"
dir "/usr/redis/data"
################################ REDIS CLUSTER  ###############################
cluster-enabled yes
cluster-node-timeout 15000
cluster-config-file nodes-6379.conf
cluster-require-full-coverage no
```

为了方便，使用sed命令创建其他配置文件

```shell
[root@localhost redis-cluster]# sed 's/6379/6380/g' redis-6379.config > redis-6380.config
```

启动节点，使用`ps`命令查看节点启动状况

```shell
[root@localhost config]# redis-server redis-6379.config
[root@localhost config]# redis-server redis-6380.config
[root@localhost config]# ps aux | grep redis-server
root      ......     Ssl  00:39   0:00 redis-server *:6379 [cluster]
root      ......     Rsl  00:40   0:00 redis-server *:6380 [cluster]
root      ......     R+   00:40   0:00 grep --color=auto redis-server
```

使用`redis-cli`命令进入redis命令行界面，添加一个数据进行测试，如下，可以发现，提示错误` CLUSTERDOWN Hash slot not served(没有提供散列槽)`。

```shell
[root@localhost config]# redis-cli -p 6379
127.0.0.1:6379> set hello world
(error) CLUSTERDOWN Hash slot not served
```

进入data数据目录，生成了日志文件以及节点配置文件

```shell
[root@localhost data]# ll
总用量 16
-rw-r--r--. 1 root root 1458 11月 15 00:39 6379.log
-rw-r--r--. 1 root root 1458 11月 15 00:40 6380.log
-rw-r--r--. 1 root root  114 11月 15 00:39 nodes-6379.conf
-rw-r--r--. 1 root root  114 11月 15 00:40 nodes-6380.conf
```

查看节点信息：

```shell
[root@localhost data]# cat nodes-6379.conf
8c38f016abedcc1da124334222082b49a30afead :0@0 myself,master - 0 0 0 connected
vars currentEpoch 0 lastVoteEpoch 0
[root@localhost data]# redis-cli -p 6379 cluster nodes
8c38f016abedcc1da124334222082b49a30afead :6379@16379 myself,master - 0 0 0 connected
[root@localhost data]# redis-cli -p 6379 cluster info
cluster_state:fail
cluster_slots_assigned:0
cluster_slots_ok:0
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:1
cluster_size:0
cluster_current_epoch:0
cluster_my_epoch:0
cluster_stats_messages_sent:0
cluster_stats_messages_received:0
```

说明：

`cluster_state:fail`：集群状态：失败

`cluster_slots_assigned:0`：集群已分配槽：0

`cluster_slots_ok:0`：集群已分配成功槽：0

`cluster_slots_pfail:0`：

`cluster_slots_fail:0`：

`cluster_known_nodes:1`：

`cluster_size:0`：

`cluster_current_epoch:0`：

`cluster_my_epoch:0`：

`cluster_stats_messages_sent:0`：

`cluster_stats_messages_received:0`：



配置说明：

```shell
################################## NETWORK #####################################
port 6379
################################# GENERAL #####################################
# 默认情况下，Redis不作为守护进程运行。如果你需要的话，用yes。
# 注意，Redis将在/var/run/ redisdis中编写一个pid文件。当监控pid。
daemonize no

# 指定日志文件名。空字符串还可以用来强制Redis登录标准输出。 
# 请注意，如果您使用标准输出进行日志记录，但是使用守护进程，日志将被发送到/dev/null。
logfile ""
################################ SNAPSHOTTING  ################################
# 要转储数据库的文件名。
dbfilename dump.rdb
# 工作目录。
# DB将被写入到这个目录中，上面使用dbfilename配置指令指定文件名。
# 仅追加的文件也将在此目录中创建。
# 注意，这里必须指定一个目录，而不是文件名。
dir ./
################################ REDIS CLUSTER  ###############################
cluster-enabled yes
cluster-node-timeout 15000
cluster-config-file nodes-6379.conf
cluster-require-full-coverage yes
```



#### 节点握手

节点握手命令：

```shell
redis-cli -p [port] cluster meet [ip] [port]
```

例如：

```shell
redis-cli -p 6349 cluster meet 192.168.56.101 6380
```

将192.168.56.2的6379和6380端口的redis服务实例进行握手：

```shell
[root@localhost config]# redis-cli -p 6379 cluster meet 192.168.56.2 6380
OK
```

再使用cluster nodes命令查看集群节点信息，信息如下：

```shell

[root@localhost config]# redis-cli -p 6379 cluster nodes
21e657a0a0b199d9d77ce7048a95314582129bc8 192.168.56.2:6380@16380 master - 0 1542231503754 1 connected
8c38f016abedcc1da124334222082b49a30afead 192.168.56.2:6379@16379 myself,master - 0 0 0 connected
[root@localhost config]# redis-cli -p 6380 cluster nodes
8c38f016abedcc1da124334222082b49a30afead 192.168.56.2:6379@16379 master - 0 1542231409600 0 connected
21e657a0a0b199d9d77ce7048a95314582129bc8 192.168.56.2:6380@16380 myself,master - 0 0 1 connected
```

在之前准备节点的时候我们也进行了集群节点信息的查看，可以发现明显的差异。这里重新显示一下：

```shell
[root@localhost data]# redis-cli -p 6379 cluster nodes
8c38f016abedcc1da124334222082b49a30afead :6379@16379 myself,master - 0 0 0 connected
```

可以看到，在没有握手之前，集群的每个节点都是孤岛状态，握手之后，它们相互之间都知道对方的存在了。

依次类推，将其它节点都进行握手：

```shell
[root@localhost config]# redis-cli -p 6379 cluster meet 192.168.56.4 6380
OK
[root@localhost config]# redis-cli -p 6379 cluster meet 192.168.56.4 6379
OK
[root@localhost config]# redis-cli -p 6379 cluster meet 192.168.56.5 6379
OK
[root@localhost config]# redis-cli -p 6379 cluster meet 192.168.56.5 6380
OK
```

再次查看集群节点信息，六个节点都进行握手了：

```shell
[root@localhost config]# redis-cli -p 6379 cluster nodes
d3b3bfd3070a371072b1060944c18f80382e61cc 192.168.56.5:6380@16380 master - 0 1542231867692 0 connected
21e657a0a0b199d9d77ce7048a95314582129bc8 192.168.56.2:6380@16380 master - 0 1542231866000 1 connected
775901c1fd24a14bf0f4c4ced15899e4e0183320 192.168.56.4:6380@16380 master - 0 1542231865000 2 connected
c21d8bd0a64076dad3341bfd53f7b3fead8aa4c7 192.168.56.4:6379@16379 master - 0 1542231866000 5 connected
9c109e6c9144ee7ffa5832bd06e5dc1aa2a76b54 192.168.56.5:6379@16379 master - 0 1542231866688 4 connected
8c38f016abedcc1da124334222082b49a30afead 192.168.56.2:6379@16379 myself,master - 0 1542231867000 3 connected
```



```shell
[root@localhost redis-cluster]# redis-cli -p 6379 cluster info
cluster_state:fail
cluster_slots_assigned:0
cluster_slots_ok:0
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:0
cluster_current_epoch:5
cluster_my_epoch:4
cluster_stats_messages_ping_sent:192
cluster_stats_messages_pong_sent:203
cluster_stats_messages_meet_sent:3
cluster_stats_messages_sent:398
cluster_stats_messages_ping_received:201
cluster_stats_messages_pong_received:195
cluster_stats_messages_meet_received:2
cluster_stats_messages_received:398
```





#### 分配槽

分配槽使用如下命令：

```shell
redis-cli -p [port] cluster addslots [slot]
```

例如：

```shell
redis-cli -p 6379 cluster addslots 0
```

每次只能分配一个槽，但是我们有16383个操作，所以编写一个脚本进行设置：

```shell
# 创建名为addslots.sh的脚本
touch addslots.sh
# 编辑
vim addslots.sh
# 设置执行权限
chmod 744 addslots.sh
```

脚本内容如下：

```shell
start=$1
end=$2
port=$3
for slot in `seq ${start} ${end}`
do
	echo "slot:${slot}"
	redis-cli -p ${port} cluster addslots ${slot}
done
```

脚本创建好之后就开始进行槽的分配，6台redis实例，三个master，总共16384个槽，所以平均分配如下：

16384/3

- `192.168.56.2:6379` -> 0~5461

- `192.168.56.4:6379` -> 5462~10922

- `192.168.56.5:6379` -> 10923~16383 

首先分配`192.168.56.2:6379`:

```shell
[root@localhost config]# sh addslots.sh 0 5461 6379
slot:0
OK
......
slot:5461
OK
```

分配完成之后查看集群节点的信息，可以发现`192.168.56.2:6379`节点已经分配了槽`0-5461`：

```shell
[root@localhost config]# redis-cli -p 6379 cluster nodes
d3b3bfd3070a371072b1060944c18f80382e61cc 192.168.56.5:6380@16380 master - 0 1542232731276 0 connected
21e657a0a0b199d9d77ce7048a95314582129bc8 192.168.56.2:6380@16380 master - 0 1542232732000 1 connected
775901c1fd24a14bf0f4c4ced15899e4e0183320 192.168.56.4:6380@16380 master - 0 1542232731000 2 connected
c21d8bd0a64076dad3341bfd53f7b3fead8aa4c7 192.168.56.4:6379@16379 master - 0 1542232729000 5 connected
9c109e6c9144ee7ffa5832bd06e5dc1aa2a76b54 192.168.56.5:6379@16379 master - 0 1542232732280 4 connected
8c38f016abedcc1da124334222082b49a30afead 192.168.56.2:6379@16379 myself,master - 0 1542232730000 3 connected 0-5461
```

依次类推，再次分配另外两个master节点的槽，如下：

```shell
[root@localhost config]# sh addslots.sh 5462 10922 6379
slot:5462
OK
slot:10922
OK

[root@localhost config]# sh addslots.sh 10923 16383 6379
slot:10923
OK
slot:16383
OK
```

查看集群节点的info信息：

```shell
[root@localhost config]# redis-cli -p 6379 cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:5
cluster_my_epoch:3
cluster_stats_messages_ping_sent:1482
cluster_stats_messages_pong_sent:1501
cluster_stats_messages_meet_sent:5
cluster_stats_messages_sent:2988
cluster_stats_messages_ping_received:1501
cluster_stats_messages_pong_received:1487
cluster_stats_messages_received:2988
```

`cluster_state:ok`	集群状态是ok的

`cluster_slots_assigned:16384`	已分配了16384 个槽

`cluster_slots_ok:16384` 	分配成功16384 个槽

`cluster_known_nodes:6` 		集群节点总共6个

`cluster_size:3` 	分配槽的结点3个

#### 分配主从

分配主从的命令如下，注意`redis-cli -p [port]`为slave节点，`[node_id]`为master节点。

```shell
redis-cli -p [port] cluster replicate [node_id]
```

在分配主从之前，我们先查看一下集群节点的信息：

```shell
[root@localhost config]# redis-cli -p 6379 cluster nodes
d3b3bfd3070a371072b1060944c18f80382e61cc 192.168.56.5:6380@16380 master - 0 1542233851279 0 connected
21e657a0a0b199d9d77ce7048a95314582129bc8 192.168.56.2:6380@16380 master - 0 1542233851000 1 connected
775901c1fd24a14bf0f4c4ced15899e4e0183320 192.168.56.4:6380@16380 master - 0 1542233850273 2 connected
c21d8bd0a64076dad3341bfd53f7b3fead8aa4c7 192.168.56.4:6379@16379 master - 0 1542233849268 5 connected 5462-10922
9c109e6c9144ee7ffa5832bd06e5dc1aa2a76b54 192.168.56.5:6379@16379 master - 0 1542233849000 4 connected 10923-16383
8c38f016abedcc1da124334222082b49a30afead 192.168.56.2:6379@16379 myself,master - 0 1542233850000 3 connected 0-5461
```

然后进行分配，三台机器，结构为错位分配，如下：

```
192.168.56.2:6379 —— 192.168.56.4:6380

192.168.56.4:6379 —— 192.168.56.5:6380

192.168.56.5:6379 —— 192.168.56.2:6380
```

开始分配

```shell
[root@localhost config]# redis-cli -p 6380 cluster replicate 9c109e6c9144ee7ffa5832bd06e5dc1aa2a76b54
OK
[root@localhost config]# redis-cli -p 6380 cluster replicate 8c38f016abedcc1da124334222082b49a30afead
OK
[root@localhost config]# redis-cli -p 6380 cluster replicate c21d8bd0a64076dad3341bfd53f7b3fead8aa4c7
OK
```

查看集群节点信息，有对应slave节点的信息：

```shell
[root@localhost config]# redis-cli -p 6379 cluster nodes
d3b3bfd3070a371072b1060944c18f80382e61cc 192.168.56.5:6380@16380 slave c21d8bd0a64076dad3341bfd53f7b3fead8aa4c7 0 1542234369068 5 connected
21e657a0a0b199d9d77ce7048a95314582129bc8 192.168.56.2:6380@16380 slave 9c109e6c9144ee7ffa5832bd06e5dc1aa2a76b54 0 1542234369000 4 connected
775901c1fd24a14bf0f4c4ced15899e4e0183320 192.168.56.4:6380@16380 slave 8c38f016abedcc1da124334222082b49a30afead 0 1542234369000 3 connected
c21d8bd0a64076dad3341bfd53f7b3fead8aa4c7 192.168.56.4:6379@16379 master - 0 1542234369000 5 connected 5462-10922
9c109e6c9144ee7ffa5832bd06e5dc1aa2a76b54 192.168.56.5:6379@16379 master - 0 1542234370072 4 connected 10923-16383
8c38f016abedcc1da124334222082b49a30afead 192.168.56.2:6379@16379 myself,master - 0 1542234368000 3 connected 0-5461
```

查看集群slots的信息：

```shell
[root@localhost config]# redis-cli -p 6379 cluster slots
1) 1) (integer) 5462
   2) (integer) 10922
   3) 1) "192.168.56.4"
      2) (integer) 6379
      3) "c21d8bd0a64076dad3341bfd53f7b3fead8aa4c7"
   4) 1) "192.168.56.5"
      2) (integer) 6380
      3) "d3b3bfd3070a371072b1060944c18f80382e61cc"
2) 1) (integer) 10923
   2) (integer) 16383
   3) 1) "192.168.56.5"
      2) (integer) 6379
      3) "9c109e6c9144ee7ffa5832bd06e5dc1aa2a76b54"
   4) 1) "192.168.56.2"
      2) (integer) 6380
      3) "21e657a0a0b199d9d77ce7048a95314582129bc8"
3) 1) (integer) 0
   2) (integer) 5461
   3) 1) "192.168.56.2"
      2) (integer) 6379
      3) "8c38f016abedcc1da124334222082b49a30afead"
   4) 1) "192.168.56.4"
      2) (integer) 6380
      3) "775901c1fd24a14bf0f4c4ced15899e4e0183320"
```

集群配置完成，测试：

报出了错误信息，错误信息是因为集群节点的实例是受保护模式的，并且绑定的地址是127.0.0.1，导致外部不能访问。按照给出的错误信息的解决方式解决即可。

```shell
[root@localhost config]# redis-cli -c -p 6379
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> set myHello world
-> Redirected to slot [8120] located at 192.168.56.4:6379
(error) DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.
```

翻译如下：

Redis是在保护模式下运行的，因为保护模式是启用的，没有指定绑定地址，也没有向客户机请求身份验证密码。在这种模式下，连接只能从环回接口接受。如果您想从外部计算机连接到Redis，您可以采用以下一种解决方案:

1. 只是禁用保护模式，发送命令`'CONFIG SET protected-mode no'`从环回接口连接到Redis，从同一主机服务器上运行，然而，如果你这样做，请确保Redis不能从互联网上公开访问。使用CONFIG重写使此更改永久性。
2. 或者，您可以通过编辑Redis配置文件来禁用受保护模式，并将受保护模式选项设置为“`no`”，然后重新启动服务器。
3. 如果您只是为了测试而手动启动服务器，请使用`'--protected-mode no'`选项重新启动服务器。
4. 设置绑定地址或身份验证密码。

注意:为了让服务器开始从外部接受连接，您只需要执行上述操作之一。



如此，使用原生命令安装的方式到此就算安装完成了。到了这里，就应该能够发现，如此配置方式即繁琐，又容易出错，所以推荐使用官方工具安装。但是用于理解redis集群是非常好的方式。

### 官方工具安装
Redis提供了一个集群命令工具`redis-trib.rb`，使用此工具可以安装集群。

命令如下：

```shell
redis-trib.rb create --replicas <arg> host1:port1 ... hostN:portN
```

其中`host:port`指的是集群的结点，`--replicas <arg>`指定的是集群每个主节点从节点的数量。

创建集群：

```shell
redis-trib.rb create --replicas 1 192.168.56.2:6379 192.168.56.2:6380 192.168.56.4:6379 192.168.56.4:6380 192.168.56.5:6379 192.168.56.5:6380
```

会提示如下错误：

```shell
/usr/bin/env: ruby: 没有那个文件或目录
```

这是因为`redis-trib.rb`是一个Ruby程序，依赖Ruby环境，所以需要安装Ruby环境。

#### 安装Ruby

安装ruby：(make && make install的过程有点长)

```shell
[root@localhost ruby]# wget https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.1.tar.gz
[root@localhost ruby]# tar -zxvf ruby-2.5.1.tar.gz
[root@localhost ruby]# cd ruby-2.5.1
[root@localhost ruby-2.5.1]# ./configure
[root@localhost ruby-2.5.1]# make && make install
[root@localhost ruby]# ruby -v
ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-linux]
```

然后执行`ruby -v`命令查看Ruby是否安装成功：

```shell
[root@localhost ruby]# ruby -v
ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-linux]
```

再次创建集群：

```shell
[root@localhost ruby]# redis-trib.rb create --replicas 1 192.168.56.2:6379 192.168.56.2:6380 192.168.56.4:6379 192.168.56.4:6380 192.168.56.5:6379 192.168.56.5:6380
Traceback (most recent call last):
	2: from /usr/redis/redis-4.0.11/src/redis-trib.rb:25:in `<main>'
	1: from /usr/local/lib/ruby/2.5.0/rubygems/core_ext/kernel_require.rb:59:in `require'
/usr/local/lib/ruby/2.5.0/rubygems/core_ext/kernel_require.rb:59:in `require': cannot load such file -- redis (LoadError)
```

可以看到还是不能创建集群，提示错误信息：“`无法加载此类文件--redis`”。这因为`redis-trib.rb`还依赖Ruby的Redis模块。

#### 安装Gem-Redis

安装Ruby的Redis模块：

```shell
[root@localhost ruby]# gem install redis
```

但是出现了如下错误：

```shell
[root@localhost ruby]# gem install redis
ERROR:  Loading command: install (LoadError)
	cannot load such file -- zlib
ERROR:  While executing gem ... (NoMethodError)
    undefined method `invoke_with_build_args' for nil:NilClass
```

##### 补救：安装zlib库

提示错误：“`无法加载此类文件—zlib`”，这是因为缺少zlib依赖，需要安装zlib库

```shell
[root@localhost ruby]# yum install -y zlib-devel
```

再次执行`gem install redis`，还是提示错误：“`无法加载此类文件—zlib`”，这是因为虽然我们安装了zlib库，但是没有集成到ruby。

集成`zlib`库到ruby：

```shell
[root@localhost ruby]# cd /usr/ruby/ruby-2.5.1/ext/zlib/
[root@localhost zlib]# ruby extconf.rb
```

编译安装：

```shell
[root@localhost zlib]# make && make install
make: *** 没有规则可以创建“zlib.o”需要的目标“/include/ruby.h”。 停止。
```

编译安装失败，发现提示错误“创建“`zlib.o”需要的目标“/include/ruby.h`”，这是因为`/usr/ruby/ruby-2.5.1/ext/zlib/`目录下的Makefile文件中的`$(top_srcdir)`变量指定的是空（`“”`），找不到对应的文件`$(top_srcdir)/include/ruby.h`，而ruby.h文件在`/usr/ruby/ruby-2.5.1/include/ruby.h`目录下。解决这个问题有三种方式：

1. 将`ruby.h`文件拷贝至`/include/ruby.h`。

   ```shell
   $ cd /
   $ mkdir include
   $ cp /usr/ruby/ruby-2.5.1/include/ruby.h /include
   ```

2. 根据目录结构，修改为相对路径，`$(top_srcdir)`修改为`../..`。

   ```shell
   zlib.o: $(top_srcdir)/include/ruby.h
   修改为
   zlib.o: ../../include/ruby.h
   ```

3. 在Makefile文件顶部声明变量`top_srcdir`指明其路径。

   ```shell
   top_srcdir=/usr/ruby/ruby-2.5.1
   ```

上述三种方式任选一种（推荐第一种），再次编译：成功

```shell
[root@localhost zlib]# make && make install
/usr/bin/install -c -m 0755 zlib.so /usr/local/lib/ruby/site_ruby/2.5.0/x86_64-linux
```

再次执行`gem install redis`，发现缺少`openssl`库：

```shell
[root@localhost ruby-2.5.1]# gem install redis
ERROR:  While executing gem ... (Gem::Exception)
    Unable to require openssl, install OpenSSL and rebuild Ruby (preferred) or use non-HTTPS sources
```

与之前`zlib`库一样，需要安装并集成，并且也具有与`zlib`一样的问题。

##### 补救：安装openssl库

安装`openssl`库：

```shell
[root@localhost ruby]# yum install openssl-devel
```

集成并编译

```shell
[root@localhost openssl]# cd /usr/ruby/ruby-2.5.1/ext/openssl
[root@localhost openssl]# ruby extconf.rb
[root@localhost openssl]# make && make install
......
/usr/bin/install -c -m 0755 openssl.so /usr/local/lib/ruby/site_ruby/2.5.0/x86_64-linux
installing default openssl libraries
```

再次执行`gem install redis`，成功了。

```shell
[root@localhost openssl]# gem install redis
Fetching: redis-4.0.3.gem (100%)
Successfully installed redis-4.0.3
Parsing documentation for redis-4.0.3
Installing ri documentation for redis-4.0.3
Done installing documentation for redis after 1 seconds
1 gem installed
```

> 注意，上面之所推荐使用第一种方式，是因为虽然在集成zlib的时候如果使用相对路径，只需要修改一处即可，但是在集成`openssl`的时候需要修改十几处。即使使用声明变量的方式也需要在zlib和openssl的Makefile文件都声明，而使用第一种拷贝`ruby.h`文件到根`/include`目录，则都可以使用。

#### 构建集群

redis-trib.rb脚本相关的环境依赖安装成功之后就可以使用redis-trib.rb脚本创建集群了，但是要注意，使用redis-trib.rb脚本创建集群还是跟前面的原生命令安装一样，先要准备好节点。有一点需要注意，如果所有的结点都在同一台机器上，就可以直接使用redis-trib.rb脚本创建集群了，但是如果有节点在不同的机器上，就需要注意节点使用有密码，使用是受保护的模式，使用绑定的地址为127.0.0.1，否则就不能连接，如下：

```shell
[root@localhost openssl]# redis-trib.rb create --replicas 1 192.168.56.2:6379 192.168.56.2:6380 192.168.56.4:6379 192.168.56.4:6380 192.168.56.5:6379 192.168.56.5:6380
>>> Creating cluster
[ERR] Sorry, can't connect to node 192.168.56.2:6379
```

> redis默认只允许本地访问，要使redis可以远程访问可以修改redis.conf，
>
> 解决办法：注释掉`bind 127.0.0.1`可以使所有的ip访问redis。
>
> 若是想指定多个ip访问，但并不是全部的ip访问，可以bind。
>
> 在redis3.2之后，redis增加了`protected-mode`，在这个模式下，即使注释掉了`bind 127.0.0.1`，再访问redisd时候还是报错
> 修改办法：`protected-mode no`

执行成功之后

```shell
[root@localhost src]# redis-trib.rb create --replicas 1 192.168.56.2:6379 192.168.56.2:6380 192.168.56.4:6379 192.168.56.4:6380 192.168.56.5:6379 192.168.56.5:6380
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

#### 测试

使用`redis-cli`登入Redis客户端，会有如下错误提示

```shell
[root@localhost src]# ./redis-cli
127.0.0.1:6379> setex name 10 redis
(error) MOVED 5798 192.168.56.4:6379
```

集群模式下登入redis-cli客户端，需要使用`-c`参数指定为集群模式，如下：

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

#### 总结

最后总结一下

```shell
# 先确认环境是否已经安装zlib和openssl库，如果没有安装，则安装
$ yum install -y zlib-devel
$ yum install -y openssl-devel
# 下载ruby压缩安装包
$ wget https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.1.tar.gz
# 解压
$ tar -zxvf ruby-2.5.1.tar.gz
# 配置
$ cd ruby-2.5.1
$ ./configure
# 编译即安装
$ make && make install
# 测试
$ ruby -v
# 安装gem-redis
$ gem install redis
# 如此redis-trib.rb脚本依赖的环境就算安装完成了，注意：这里没有集成“zlib”以及“openssl”到Ruby，
# 是因为在安装ruby之前已经安装了“zlib”和“openssl”库，编译时这些工作都做了。
# 上述的集成操作更多的是对安装编译Ruby之前没有安装“zlib”和“openssl”库的补救措施。
# 构建Redis集群
$ redis-trib.rb create --replicas <slave-number> [ip]:[port] [ip]:[port] [ip]:[port]...
```



> 配置集群只需要配置一次，再次启动时只需要启动其集群节点服务器就可以了，不需要在使用`redis-trib.rb`脚本启动集群，因为在首次启动集群时会在data/目录下成集群配置文件，如下：
>
> ```shell
> [root@localhost data]# cd /usr/redis/redis-cluster/data | ll
> 总用量 26132
> ......
> -rw-r--r--. 1 root root      799 12月  3 01:08 nodes-6379.conf
> -rw-r--r--. 1 root root      799 12月  3 01:08 nodes-6380.conf
> ```
>
> 再次启动集群节点时会找到这个集群配置文件，加载相应的信息。



```shell
[root@localhost redis]# redis-trib.rb create --replicas 1 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386
WARNING: redis-trib.rb is not longer available!
You should use redis-cli instead.

All commands and features belonging to redis-trib.rb have been moved
to redis-cli.
In order to use them you should call redis-cli with the --cluster
option followed by the subcommand name, arguments and options.

Use the following syntax:
redis-cli --cluster SUBCOMMAND [ARGUMENTS] [OPTIONS]

Example:
redis-cli --cluster create 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 --cluster-replicas 1

To get help about all subcommands, type:
redis-cli --cluster help

[root@localhost redis]# redis-cli --cluster create 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 --cluster-replicas 1
[ERR] Node 127.0.0.1:6381 NOAUTH Authentication required.
[root@localhost redis]# redis-cli --cluster create 127.0.0.1:6381 -a 55a41794b45c4e9b9e87c5091df3be90 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 --cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
[ERR] Wrong number of arguments for specified --cluster sub command
[root@localhost redis]# redis-cli -a 55a41794b45c4e9b9e87c5091df3be90 --cluster create 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 --cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:6384 to 127.0.0.1:6381
Adding replica 127.0.0.1:6385 to 127.0.0.1:6382
Adding replica 127.0.0.1:6386 to 127.0.0.1:6383
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 0aa770cb94c2e48c02f3caaff0b4c782699c6ee2 127.0.0.1:6381
   slots:[0-5460] (5461 slots) master
M: ddbf87e6674a8a4f7e5cb4fc430fb2e056b2f39e 127.0.0.1:6382
   slots:[5461-10922] (5462 slots) master
M: 20ef4b540a3f5eab832b8a899cb0633db825ead2 127.0.0.1:6383
   slots:[10923-16383] (5461 slots) master
S: 3fb8b0349cfdd8042e3b65431d66d6f5ceef1388 127.0.0.1:6384
   replicates ddbf87e6674a8a4f7e5cb4fc430fb2e056b2f39e
S: 77443eb0067c9016daf5c6f586cda04829c038f9 127.0.0.1:6385
   replicates 20ef4b540a3f5eab832b8a899cb0633db825ead2
S: e227c18a5ceba412aa5e071a7cebeeea947ac100 127.0.0.1:6386
   replicates 0aa770cb94c2e48c02f3caaff0b4c782699c6ee2
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
......
>>> Performing Cluster Check (using node 127.0.0.1:6381)
M: 0aa770cb94c2e48c02f3caaff0b4c782699c6ee2 127.0.0.1:6381
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 77443eb0067c9016daf5c6f586cda04829c038f9 127.0.0.1:6385
   slots: (0 slots) slave
   replicates 20ef4b540a3f5eab832b8a899cb0633db825ead2
S: e227c18a5ceba412aa5e071a7cebeeea947ac100 127.0.0.1:6386
   slots: (0 slots) slave
   replicates 0aa770cb94c2e48c02f3caaff0b4c782699c6ee2
S: 3fb8b0349cfdd8042e3b65431d66d6f5ceef1388 127.0.0.1:6384
   slots: (0 slots) slave
   replicates ddbf87e6674a8a4f7e5cb4fc430fb2e056b2f39e
M: ddbf87e6674a8a4f7e5cb4fc430fb2e056b2f39e 127.0.0.1:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 20ef4b540a3f5eab832b8a899cb0633db825ead2 127.0.0.1:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



redis安装
make[3]: gcc：命令未找到

Linux系统初始工具安装
yum install -y wget
yum install -y gcc
yum install -y vim
yum install -y zlib-devel
yum install -y openssl-devel
软连接

实体vo dto

sql分组