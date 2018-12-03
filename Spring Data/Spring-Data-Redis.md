






## Features

- 连接包作为跨多个Redis驱动程序/连接器的低级抽象 ([Jedis](https://github.com/xetorthio/jedis)和[Lettuce](https://github.com/mp911de/lettuce)。不支持 [JRedis](https://github.com/alphazero/jredis) 和[SRP](https://github.com/spullara/redis-protocol)。)

- 针对Redis驱动程序异常，将异常转换为Spring的可移植数据访问异常层次结构
- 为执行各种Redis操作、异常转换和序列化支持提供高级抽象的`RedisTemplate`
- [Pubsub](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#pubsub)支持（例如消息驱动pojo的`MessageListenerContainer`）
- [Redis Sentinel](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:sentinel) and [Redis Cluster](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#cluster) support
- JDK, String, JSON 和 Spring Object/XML 映射[序列化器](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:serializer)
- JDK集合实现在Redis之上
- 原子[计数器](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:support)类支持
- `Sorting`和`Pipelining`功能
- 专门支持`SORT`，`SORT/GET`模式和返回的批量值
- Spring 3.1缓存抽象的Redis实现
- `Repository`接口的自动实现，包括对使用`@EnableRedisRepositories`的自定义查找器方法的支持
- 对存储库的CDI支持



## [1. New Features](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#new-features)





## [2. Why Spring Data Redis?](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#why-spring-redis)



## [3. Requirements](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#requirements)

Spring Data Redis 2.x binaries(二进制文件) require JDK level 8.0 and above(and above：及以上)和Spring Framework 5.1.3.RELEASE and above。

在键值存储方面，需要Redis 2.6.x或者更高的版本，Spring Data Redis目前针对最新的4.0版本进行了测试。

## [4. Getting Started](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#get-started)

本节提供了一个易于遵循的指南，用于开始使用Spring Data Redis模块。

### [4.1. First Steps](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#get-started:first-steps)

正如[Why Spring Data Redis?](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#why-spring-redis)解释的那样，Spring Data Redis (SDR)提供了Spring框架和Redis键值存储之间的集成。因此，您应该熟悉这两个框架。在整个SDR文档中，每个部分都提供了相关资源的链接。但是，在阅读本指南之前，您应该熟悉这些主题。

## [Reference Documentation](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#reference)

## [5. Redis support](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis)

Spring数据支持的键值存储之一是Redis。引用Redis项目主页:

Spring Data Redis提供了来自Spring应用程序的简单配置和对Redis的访问。它提供了与存储交互的低级和高级抽象，使用户不必担心基础结构问题。

### [5.1. Redis Requirements](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:requirements)

Spring Redis需要Redis 2.6或更高版本，Spring Data Redis集成了[Lettuce](https://github.com/lettuce-io/lettuce-core)和[Jedis](https://github.com/xetorthio/jedis)，这是两个流行的用于Redis的开源Java库。

### [5.2. Redis Support High-level View](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:architecture)

Redis支持提供了几个组件。对于大多数任务，高级抽象和支持服务是最佳选择。注意，在任何一点上，您都可以在层之间移动。例如，您可以获得一个低级连接(甚至是本机库)来直接与Redis通信。





- [5.3. Connecting to Redis](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:connectors)
  - [5.3.1. RedisConnection and RedisConnectionFactory](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:connectors:connection)
  - [5.3.2. Configuring the Lettuce Connector](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:connectors:lettuce)
  - [5.3.3. Configuring the Jedis Connector](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:connectors:jedis)
  - [5.3.4. Write to Master, Read from Replica](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:write-to-master-read-from-replica)
- [5.4. Redis Sentinel Support](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:sentinel)
- [5.5. Working with Objects through RedisTemplate](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:template)
- [5.6. String-focused Convenience Classes](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:string)
- [5.7. Serializers](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:serializer)
- [5.8. Hash mapping](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.hashmappers.root)
  - [5.8.1. Hash Mappers](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_hash_mappers)
  - [5.8.2. Jackson2HashMapper](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.hashmappers.jackson2)
- [5.9. Redis Messaging (Pub/Sub)](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#pubsub)
  - [5.9.1. Publishing (Sending Messages)](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:pubsub:publish)
  - [5.9.2. Subscribing (Receiving Messages)](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:pubsub:subscribe)
- [5.10. Redis Transactions](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#tx)
  - [5.10.1. @Transactional Support](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#tx.spring)
- [5.11. Pipelining](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#pipeline)
- [5.12. Redis Scripting](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#scripting)
- [5.13. Support Classes](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:support)
  - [5.13.1. Support for the Spring Cache Abstraction](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:support:cache-abstraction)







## [6. Reactive Redis support](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis:reactive)



## [7. Redis Cluster](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#cluster)

使用Redis集群需要Redis Server version 3.0+。有关更多信息，请参阅集群教程。

### [7.1. Enabling Redis Cluster](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_enabling_redis_cluster)

集群支持与非集群通信基于相同的构建块。RedisClusterConnection是RedisConnection的扩展，它处理与Redis集群的通信，并将错误转换为Spring DAO异常层次结构。RedisClusterConnection实例是用RedisConnectionFactory创建的，RedisConnectionFactory必须使用关联的RedisClusterConfiguration进行设置，如下面的示例所示:

Example 3.  RedisConnectionFactory配置Redis集群的例子：





> `RedisClusterConfiguration`也可以通过`PropertySource`定义，它具有以下属性:
>
> Configuration Properties
>
> - `spring.redis.cluster.nodes`: 逗号分隔的主机列表--host:port对。
> - `spring.redis.cluster.max-redirects`: 允许的群集重定向数目。

> 初始配置将驱动程序库指向集群节点的初始集。动态集群重新配置所产生的更改仅保存在本地驱动程序中，不会写回配置。

### [7.2. Working With Redis Cluster Connection](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_working_with_redis_cluster_connection)

正如前面提到的，Redis集群的行为不同于单节点Redis，甚至不同于哨兵监视的主副本环境。这是因为自动分片将键映射到分布在节点上的16384个槽中的一个。因此，涉及多个键的命令必须断言所有键映射到完全相同的槽，以避免跨槽执行错误。单个集群节点只提供一组专用密钥。针对一个特定服务器发出的命令只返回该服务器提供的那些键的结果。作为一个简单的例子，考虑KEYS命令。当发送到集群环境中的服务器时，它只返回请求发送到的节点所提供的键，不一定返回集群中的所有键。因此，要获取集群环境中的所有键，必须从所有已知的主节点读取键。

虽然将特定键重定向到相应的槽服务节点是由驱动程序库处理的，但是RedisClusterConnection覆盖了更高级的功能，例如跨节点收集信息或向集群中的所有节点发送命令。拾取前面的键示例，这意味着keys(pattern)方法拾取集群中的每个主节点，同时在每个主节点上执行keys命令，同时拾取结果并返回累积的键集。仅仅请求单个节点的键，RedisClusterConnection就会为这些方法(例如，键(节点，模式))提供重载。

可以从`RedisClusterConnection`中获得`RedisClusterNode`。或者它可以通过使用主机和端口或节点Id来构造。

下面的示例显示了在集群中运行的一组命令:

*Example 4. Sample of Running Commands Across the Cluster*

```text
redis-cli@127.0.0.1:7379 > cluster nodes

6b38bb... 127.0.0.1:7379 master - 0 0 25 connected 0-5460                      （1）
7bb78c... 127.0.0.1:7380 master - 0 1449730618304 2 connected 5461-10922       （2）
164888... 127.0.0.1:7381 master - 0 1449730618304 3 connected 10923-16383      （3）
b8b5ee... 127.0.0.1:7382 slave 6b38bb... 0 1449730618304 25 connected          （4）
```



```java
RedisClusterConnection connection = connectionFactory.getClusterConnnection();

connection.set("thing1", value);                          （5）                     
connection.set("thing2", value);                          （6）                    

connection.keys("*");                                     （7）                    

connection.keys(NODE_7379, "*");                          （8）                   
connection.keys(NODE_7380, "*");                          （9）                  
connection.keys(NODE_7381, "*");                          （10）                  
connection.keys(NODE_7382, "*");                          （11）
```

1. 主节点服务插槽0到5460，复制到7382的副本
2. 主节点服务插槽5461到10922
3. 主节点服务插槽10923到16383
4. 在7379处保存主副本的副本节点
5. 请求路由到节点7381服务插槽12182
6. 请求路由到节点7379服务插槽5061
7. 请求路由到节点在7379、7379、7380 → [thing1, thing2]
8. 请求路由到节点7379 → [thing2]
9. 请求路由到节点7380→ []
10. 请求路由到节点7381 → [thing1]
11. 请求路由到节点7382 → [thing2]

当所有键映射到同一个槽时，本机驱动程序库自动服务于跨槽请求，例如MGET。但是，一旦情况不是这样，`RedisClusterConnection`将对槽服务节点执行多个并行GET命令，并再次返回累积结果。这比单槽执行的性能要差，因此应该小心使用。如果有疑问，考虑通过在大括号中提供前缀将键固定到同一个槽，例如`{my-prefix}.thing1`和`{my-prefix}.thing2`，它会映射到同一个槽号。下面的例子显示了跨槽请求处理:

*Example 5. 跨槽请求处理示例*

```text
redis-cli@127.0.0.1:7379 > cluster nodes

6b38bb... 127.0.0.1:7379 master - 0 0 25 connected 0-5460                 (1)     
7bb...
```

```java
RedisClusterConnection connection = connectionFactory.getClusterConnnection();

connection.set("thing1", value);           // slot: 12182
connection.set("{thing1}.thing2", value);  // slot: 12182
connection.set("thing2", value);           // slot:  5461

connection.mGet("thing1", "{thing1}.thing2");                             (2)     

connection.mGet("thing1", "thing2");                                      (3)
```

1. 与前面示例中的配置相同。

2. 键映射到相同的槽→127.0.0.1:7381 MGET thing1 { thing1 } .thing2

3. 键映射到不同的插槽，并被分割成单个插槽，这些插槽路由到相应的节点

   →127.0.0.1:7379 GET thing2

   →127.0.0.1:7381 GET thing1

> 前面的示例演示了Spring Data Redis遵循的一般策略。请注意，一些操作可能需要将大量数据加载到内存中来计算所需的命令。此外，并不是所有跨槽请求都可以安全地移植到多个单槽请求，如果使用不当(例如PFCOUNT)，则会出错。

### [7.3. Working with `RedisTemplate` and `ClusterOperations`](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_working_with_redistemplate_and_clusteroperations)

有关RedisTemplate的一般用途、配置和使用，请参阅RedisTemplate中对象的处理部分。

> 使用任何JSON重序列化器设置RedisTemplate#keySerializer时都要小心，因为更改JSON结构会直接影响哈希槽的计算。

RedisTemplate通过`ClusterOperations`接口提供对特定于集群的操作的访问，该接口可以从`RedisTemplate. opsforcluster()`获得。这允许您显式地在集群中的单个节点上运行命令，同时保留为模板配置的序列化和反序列化特性。它还提供管理命令(例如`CLUSTER MEET`)或更高级的操作(例如，重新分片)。

下面的例子显示了如何访问RedisClusterConnection与RedisTemplate:

Example 5. 访问RedisClusterConnection与RedisTemplate

```java
ClusterOperations clusterOps = redisTemplate.opsForCluster();
clusterOps.shutdown(NODE_7379);                                         ① 
```

在7379关闭节点，交叉手指有一个副本可以接管。

## [8. Redis Repositories](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories)

- 