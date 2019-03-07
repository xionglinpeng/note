






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

### [5.9. Redis Messaging (Pub/Sub)](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#pubsub)



#### [5.9.1. Publishing (Sending Messages)](https://docs.spring.io/spring-data/redis/docs/2.1.5.RELEASE/reference/html/#redis:pubsub:publish)



#### [5.9.2. Subscribing (Receiving Messages)](https://docs.spring.io/spring-data/redis/docs/2.1.5.RELEASE/reference/html/#redis:pubsub:subscribe)



### [5.10. Redis Transactions](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#tx)



#### [5.10.1. @Transactional Support](https://docs.spring.io/spring-data/redis/docs/2.1.5.RELEASE/reference/html/#tx.spring)





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

使用Redis存储库可以在Redis哈希中无缝转换和存储域对象，应用自定义映射策略并利用二级索引。

> Redis存储库至少需要Redis Server 2.8.0版本。

### [8.1. Usage](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.usage)

Spring Data Redis让您轻松实现域实体，如下面的示例所示:

*Example 8：Person实体示例*

```java
@Data
@RedisHash("people")
public class Person {
    @Id
    String id;
    String firstname;
    String lastname;
    Address address;
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Address {
    private String country;
    private String city;
}
```

我们有一个非常简单的域对象。注意，它的类型上有一个`@RedisHash`注解，以及一个名为`id`的属性，该属性使用`org.springframework.data.annotation.Id`进行注解。这两项负责创建用于持久化hash的实际key。

>命名id或者用@Id标注的属性被视为标识符属性。即如果实体类中具有命名为`id`的属性，那么以`id`属性作为键，如果没有命令为`id`的属性，那么以标注`@Id`的属性作为键。如果同时即有命令为`id`的属性和标注`@Id`注解的属性，那么以标注`@Id`的属性作为键。

现在要实际拥有一个负责存储和检索的组件，我们需要定义一个存储库接口，如下面的示例所示:

*Example: 持久化Persion实体的基本存储库接口*

```java
public interface PersonRepository extends CrudRepository<Person,String> {
}
```

当我们的存储库扩展CrudRepository时，它提供了基本的CRUD和finder操作。在这两者之间我们需要的是相应的Spring配置，如下例所示：

*Example 9 : JavaConfig for RedisRepositories*

```java
@Configuration
@EnableRedisRepositories
public class ApplicationConfig {

  @Bean
  public RedisConnectionFactory connectionFactory() {
    return new JedisConnectionFactory();
  }

  @Bean
  public RedisTemplate<?, ?> redisTemplate() {

    RedisTemplate<byte[], byte[]> template = new RedisTemplate<byte[], byte[]>();
    return template;
  }
}
```

根据前面的设置，我们可以将PersonRepository注入到组件中，如下面的示例所示：

*Example 10 ：Access to Persion Entities*

```java
@Autowired
private PersonRepository personRepository;

public void basicCrudOperations(){
    Person person = new Person();
    person.setFirstname("小明");
    person.setLastname("小张");
    person.setAddress(new Address("china","sichuan"));

    Person pers = personRepository.save(person);   ①

    Optional<Person> optionalPerson = personRepository.findById(person.getId());  ②

    long count = personRepository.count();  ③

    personRepository.delete(person);  ④
}
```

①

② 使用提供的id检索存储在`keyspace:id`中的对象。

③

④ 从Redis中删除给定对象的键。

[8.2. Object Mapping Fundamentals](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#mapping.fundamentals)

- [8.2.1. Object creation](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#mapping.object-creation)
- [8.2.2. Property population](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#mapping.property-population)
- [8.2.3. General recommendations](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_general_recommendations)
- [8.2.4. Kotlin support](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_kotlin_support)

### [8.3. Object-to-Hash Mapping](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.mapping)

Redis存储库支持将对象持久化为哈希。这需要通过`RedisConverter`进行对象到哈希的转换。默认实现使用转换器将属性值与Redis本地byte[]进行映射。

前几节中给定的Person类型，默认映射如下所示:

```
_class = com.xlp.example.redis.repositories.entity.Person    ①
id = 2b084711-6606-42bb-ab98-b32cdc604aca
firstname = 小明        ②
lastname = 小张
address.country = china    ③
address.city = sichuan
```

① _class属性包括在根级别以及任何嵌套接口或抽象类型上。

② 简单的属性值由路径映射。

③ 复杂类型的属性由它们的点路径映射。

下表描述了默认映射规则：

*Table 7. Default Mapping Rules*

| Type                    | Sample                                                       | Mapped Value                                                 |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 简单类型（例如String）  | String firstname = "小明"                                    | firstname="小明"                                             |
| 复杂类型（例如Address） | Address address = new Address("china", "sichuan");           | address.city="china"                                         |
| List简单类型            | List<String> nicknames = Arrays.asList("西厢记","三国霸业"); | nicknames[0]="西厢记"<br/>nicknames[1]="三国霸业"            |
| Map简单类型             | Map<String,String> atts = new HashMap<>();<br/>atts.put("name","小王"); <br/>atts.put("age","22"); | atts.[name]="小王"<br/>atts.[age]="22"                       |
| List复杂类型            | List<Address> address= Arrays.asList(new Address("china","sichuan")); | address.[0].city="sichuan"<br/>address.[1].city="beijing"    |
| Map复杂类型             | Map<String,Address> addressMap = new HashMap<>(); <br/>addressMap.put("address1",new Address("china","sichuan")); <br/>addressMap.put("address2",new Address("china","beijing")); | address.[address1].city="sichuan"<br/>address.[address2].city="beijing" |

> 由于这种平面表示结构，映射键需要是简单类型，例如`String`或`Number`。

映射行为可以通过在`RedisCustomConversions`中注册相应的`Converter`来定制。这些转换器可以处理从一个`byte[]`到另一个`byte[]`的转换，以及`Map<String,byte[]>`。第一个方法适用于(例如)将复杂类型转换为(例如)仍然使用默认映射散列结构的二进制JSON表示。第二个选项提供了对结果散列的完全控制。

> 将对象写入Redis散列将删除散列中的内容并重新创建整个散列，因此将丢失未映射的数据。

下面的例子显示了两个样本字节数组转换器:

*Exmple 13. Sample byte[] Converters*

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.xlp.example.redis.repositories.entity.Address;
import org.springframework.core.convert.converter.Converter;
import org.springframework.data.convert.WritingConverter;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.lang.NonNull;

@WritingConverter
public class AddressToBytesConverter implements Converter<Address, byte[]> {

    private final Jackson2JsonRedisSerializer<Address> serializer;

    public AddressToBytesConverter(){
        serializer = new Jackson2JsonRedisSerializer<>(Address.class);
        serializer.setObjectMapper(new ObjectMapper());
    }

    @Override
    public byte[] convert(@NonNull Address address) {
        return serializer.serialize(address);
    }
}

import com.fasterxml.jackson.databind.ObjectMapper;
import com.xlp.example.redis.repositories.entity.Address;
import org.springframework.core.convert.converter.Converter;
import org.springframework.data.convert.ReadingConverter;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.lang.NonNull;

@ReadingConverter
public class BytesToAddressConverter implements Converter<byte[], Address> {

    private final Jackson2JsonRedisSerializer<Address> serializer;

    public BytesToAddressConverter(){
        serializer = new Jackson2JsonRedisSerializer<>(Address.class);
        serializer.setObjectMapper(new ObjectMapper());
    }

    @Override
    public Address convert(@NonNull byte[] address) {
        return serializer.deserialize(address);
    }
}
```

在`org.springframework.data.redis.core.convert.RedisCustomConversions`中注册`org.springframework.core.convert.converter.Converter`：

```java
@Bean
public RedisCustomConversions redisCustomConversions(){
    List<Converter> converters = Arrays.asList(
        new AddressToBytesConverter(),
        new BytesToAddressConverter());
    return new RedisCustomConversions(converters);
}
```

使用前面的字节数组转换器产生类似如下输出：

```
_class = com.xlp.example.redis.repositories.entity.Person
id = 2b084711-6606-42bb-ab98-b32cdc604aca
firstname = 小明
lastname = 小张
address = {"country": "china","city": "sichuan"}
```

下面的例子展示了两个`Map`转换器的例子:

*Example 14. Sample Map<String,byte[]> Converters*

```java
import com.xlp.example.redis.repositories.entity.Address;
import org.springframework.core.convert.converter.Converter;
import org.springframework.data.convert.WritingConverter;
import org.springframework.lang.NonNull;

import java.util.Collections;
import java.util.Map;

@WritingConverter
public class AddressToMapConverter implements Converter<Address, Map<String,byte[]>> {

    @Override
    public Map<String, byte[]> convert(@NonNull Address address) {
        return Collections.singletonMap("ciudad",address.getCity().getBytes());
    }
}

import com.xlp.example.redis.repositories.entity.Address;
import org.springframework.core.convert.converter.Converter;
import org.springframework.data.convert.ReadingConverter;
import org.springframework.lang.NonNull;

import java.util.Map;

@ReadingConverter
public class MapToAddressConverter implements Converter<Map<String,byte[]>, Address> {

    @Override
    public Address convert(@NonNull Map<String, byte[]> source) {
        return new Address("china",new String(source.get("ciudad")));
    }
}
```

使用前面的Map`Converter`产生类似如下输出:

```
_class = com.xlp.example.redis.repositories.entity.Person
id = 2b084711-6606-42bb-ab98-b32cdc604aca
firstname = 小明
lastname = 小张
address.ciudad = sichuan
```

> 自定义转换对索引分辨率没有影响。即使对于自定义转换的类型，也会创建辅助索引。

#### [8.3.1. Customizing Type Mapping](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_customizing_type_mapping)

如果您希望避免将整个Java类名写成类型信息，而希望使用键，您可以在持久化的实体类上使用`@TypeAlias`注释。如果您需要进一步定制映射，看看`TypeInformationMapper`接口。该接口的实例可以在`DefaultRedisTypeMapper`上配置，可以在`MappingRedisConverter`上配置。

下面的例子展示了如何为实体定义类型别名:

*Example 15. Defining `@TypeAlias` for an entity*

```java
@Data
@TypeAlias("com.person")
@RedisHash(value = "people",timeToLive = 1000)
public class Person {
    
}
```

生成的hash文档将`com.person`作为`_class`字段中的值。

**Configurting Custom Type Mapping**

下面的例子演示了如何在`MappingRedisConverter`中配置自定义`RedisTypeMapper`：

*Example 16. Configuring a custom `RedisTypeMapper` via Spring Java Config*







### [8.4. Keyspaces](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.keyspaces)

Keyspaces是用于定义创建Redis Hash的键的前缀，默认情况下，前缀为持久化Hash实体类的`getClass(). getname()`，我们可以通过以下三种方式更改其默认值：

1. `@RedisHash`的`value`属性。
2. `@KeySpace`注解。
3. 编程式方式：继承`org.springframework.data.redis.core.convert.KeyspaceConfiguration`，并重写`initialConfiguration()`方法，然后集成。

注意，上述的三种配置方式都有一个优先级：`@KeySpace` > `@RedisHash(value=...)` > 编程式方式。

下面的例子展示了如何使用`@KeySpace`注解设置keyspace：

```java
@RedisHash
@KeySpace("{people}")
public class Person {
    ......
}
```

下面的例子展示了如何使用`@EnableRedisRepositories`注解设置keyspace：

*Example 17. Keyspace Setup via `@EnableRedisRepositories`*

```java
public class MyKeyspaceConfiguration extends KeyspaceConfiguration {
    @Override
    protected @NonNull Iterable<KeyspaceSettings> initialConfiguration() {
        return Collections.singleton(new KeyspaceSettings(Person.class,"myPersion"));
    }
}

@SpringBootApplication
@EnableRedisRepositories(keyspaceConfiguration = MyKeyspaceConfiguration.class)
public class RepositoriesMain {
    ......
}
```

下面的示例展示如何以编程方式设置keyspace：

*Example 18. Progeammatic Keyscace setup*

```java
public class MyKeyspaceConfiguration extends KeyspaceConfiguration {
    @Override
    protected @NonNull Iterable<KeyspaceSettings> initialConfiguration() {
        return Collections.singleton(new KeyspaceSettings(Person.class,"myPersion"));
    }
}

@Bean(name = "keyValueMappingContext")
public RedisMappingContext keyValueMappingContext(){
    return new RedisMappingContext(
        new MappingConfiguration(
            new IndexConfiguration(),
            new MyKeyspaceConfiguration()));
}
```

注意，使用这种方式，其`RedisMappingContext`bean的名称必须为`keyValueMappingContext`，否则将无效。

### [8.5. Secondary Indexes](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.indexes)

二级索引用于支持基于本地Redis结构的查找操作。值被写入每次保存的根据索引，并在对象被删除或过期时被删除。

#### [8.5.1. Simple Property Index](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.indexes.simple)

给定前面显示的示例Person实体，我们可以为firstname创建一个索引，方法是用@Indexed注释属性，如下面的例子所示:

*Example 19. Annotation driven indexing*

```java
@RedisHash(value = "people")
public class Person {
    @Id
    String id;
    @Indexed
    String firstname;
    String lastname;
    Address address;
}
```

为属性值建立索引。添加两个Person（例如“tom”和“jerry”），将会在Redis中生成如下索引：

> 注意，这个索引是set集合，通过`sadd`命令添加，即是说，同一个名字会包含多个索引。

```
SADD people:firstname:tom
	899bf3e4-9cf0-4ffd-8660-07e99cfbf091
SADD people:firstname:jerry
	e2dca935-2daa-4373-a4ec-1678bb4384ad
```

同时还会生成一下两个反向的索引（也是set集合）：

```
SADD people::899bf3e4-9cf0-4ffd-8660-07e99cfbf091:idx
	people:firstname:tom
SADD people::e2dca935-2daa-4373-a4ec-1678bb4384ad:idx
	people:firstname:jerry
```

在嵌套的对象上也可以有索引，例如在`Address`属性对象上有一个`city`属性，对该属性使用`@Indexed`进行注解，一旦当`person.address.city`属性不为空，那么就会为设置对应的二级索引，如下面的示例所示：

```
SADD people:address.city:sichuan
	899bf3e4-9cf0-4ffd-8660-07e99cfbf091
```

同样，在`idx`,`set`集合当中会包含这个嵌套对象的索引：

```
SADD people::899bf3e4-9cf0-4ffd-8660-07e99cfbf091:idx
	people:firstname:tom
	people:address.city:sichuan
```

另外，编程设置可以让我们在Map和List集合属性上定义索引，如下面的示例所示：

```java
@RedisHash(value = "people")
public class Person {
    // ... other properties omitted
    @Indexed
    Map<String,String> attributes;  ①
    @Indexed
    Map<String,Person> relatives;   ②
    @Indexed
    List<Address> addresses;        ③
}
```

① `people:attributes.age:20 899bf3e4-9cf0-4ffd-8660-07e99cfbf091`

② `people:relatives.person.firstname:spongeBob 899bf3e4-9cf0-4ffd-8660-07e99cfbf091`

③ `people:addresses.city:chengdu 899bf3e4-9cf0-4ffd-8660-07e99cfbf091 `

同样，会在`idx`的`set`集合当中会包含这写集合的索引。

> 不能在[References](https://docs.spring.io/spring-data/redis/docs/2.1.4.RELEASE/reference/html/#redis.repositories.references)上解析索引。

与keyspaces一样，您可以通过继承`org.springframework.data.redis.core.index.IndexConfiguration`类，重写`initialConfiguration()`方法指定索引属性（`@EnableRedisRepositories`的`indexConfiguration`属性指定索引索引配置类），而不用通过`@Indexed`注解注释对象域类型。如下面的示例所示:

*Example 20. Index Setup with @EnableRedisRepositories*

```java
@Configuration
@EnableRedisRepositories(indexConfiguration = MyIndexConfiguration.class)
public class ApplicationConfig {

  //... RedisConnectionFactory and RedisTemplate Bean definitions omitted

  public static class MyIndexConfiguration extends IndexConfiguration {

    @Override
    @NonNull
    protected Iterable<IndexDefinition> initialConfiguration() {
      return Collections.singleton(new SimpleIndexDefinition("people", "firstname"));
    }
  }
}
```

与keyspaces一样，可以通过编程方式配置索引，如下面的示例所示:

*Example 21. Programmatic Index setup*

```java
@Configuration
@EnableRedisRepositories
public class ApplicationConfig {

  //... RedisConnectionFactory and RedisTemplate Bean definitions omitted
    
  @Bean(name = "keyValueMappingContext")
  public RedisMappingContext keyValueMappingContext(){
      return new RedisMappingContext(
          new MappingConfiguration(
              new MyIndexConfiguration(),
              new MyKeyspaceConfiguration()));
  }
    
  public static class MyIndexConfiguration extends IndexConfiguration {

    @Override
    @NonNull
    protected Iterable<IndexDefinition> initialConfiguration() {
      return Collections.singleton(new SimpleIndexDefinition("people", "firstname"));
    }
  }
}
```

上面的方式其实就是通过代码配置的方式指定要生成索引的属性信息。

注意：通过编程的方式，优先级比注解方式，@Enable属性方式更高。

#### [8.5.2. Geospatial Index(地理空间索引)](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.indexes.geospatial)



### [8.6. Query by Example](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#query-by-example)

本章对Query by Example进行了介绍，并介绍如何使用它。

Query by Example (QBE)是一种用户友好的查询技术，具有简单的接口。它允许动态创建查询，并且不要求您编写包含字段名的查询。事实上，Query by Example根本不要求您使用特定的查询语言编写查询。

#### [8.6.1. Introduction](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#query-by-example.introduction)

Query by Example API由三部分组成：

- Probe：具有填充字段的域对象的实际示例。
- `ExampleMatcher`：`ExampleMatcher`提供关于如何匹配特定字段的详细信息。它可以跨多个示例重用。
- `Example`：一个Example由probe和`ExampleMatcher`组成。它用于创建查询。

Query by Example非常适合几个用例：

- 使用一组静态或动态约束查询数据存储。
- 频繁地重构域对象，而不用担心会破坏现有的查询。
- 独立于底层数据存储API工作。

Query by Example也有几个限制:

- 不支持嵌套或分组的属性约束，例如 `firstname = ?0 和 (firstname = ?1 and lastname = ?2)`。
- 仅支持字符串的开始/包含/结束/正则表达式匹配和其他属性类型的精确匹配。

在开始使用Query by Example之前，您需要一个域对象。首先，为您的存储库创建一个接口，如下面的示例所示：

*Example 22. Simple Person object*

```java

```



#### [8.6.2. Usage](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#query-by-example.usage)
####  [8.6.3. Example Matchers](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#query-by-example.matchers)
####  [8.6.4. Executing an Example](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#query-by-example.execution)

### [8.7. Time To Live](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.expirations)

存储在Redis中的对象可能只在一定的时间内有效。这对于在Redis中持久化短生命周期的对象特别有用，当它们到达生命尽头时，无需手动删除它们。可以使用`@RedisHash(timeToLive=…)`设置以秒为单位的过期时间，也可以使用`KeyspaceSettings`设置过期时间(参见Keyspaces)。

可以通过在数值属性或方法上使用`@TimeToLive`注释设置更灵活的过期时间。但是，不要在同一个类中的方法和属性上都应用@TimeToLive。下面的示例显示了属性和方法上的`@TimeToLive`注释:

Example 29 : Expirations

```java
@Data
@RedisHash(value = "people",timeToLive = 1000)
public class Person {
    @Id
    private String id;
    private String firstname;
    private String lastname;
    private Address address;

    @TimeToLive
    private long expiretion = 120L;

    @TimeToLive
    public long getTimeToLive() {
        long seconds = new Random().nextInt();
        System.out.printf("time to live : %ds\n",seconds);
        return seconds;
    }
}
```

注意：已知设置过期的时间的方式有三种，它们之间的优先级是:

`TimeToLive Property`> `TimeToLive Method` > `@RedisHash(timeToLive = ...)`。

> 使用@TimeToLive显式注释属性将从Redis读取实际的TTL或PTTL值。即是说，如果是使用`@TimeToLive`注解标注属性设置过期时间，那么在查询的时候，例如执行`findById`等方法的时候，查询的对象其标注`@TimeToLive`注解的属性将返回当前Hash实际还剩余的有效时间。
>
> -1表示对象没有相关的过期时间。

### [8.8. Persisting References](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.references)

用@Reference标记属性允许存储一个简单的键引用，而不是将值复制到散列本身。从Redis加载时，引用被自动解析并映射回对象中，如下例所示：

*Example 30 : Sample Perperty Reference*

```

```

### [8.9. Persisting Partial Updates](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.partial-updates)

在某些情况下，您不需要为了在整个实体中设置一个新值而加载并重写整个实体。例如更新最后一个活动时间的会话时间戳这样一种场景，可能您希望更改一个属性。`PartialUpdate`允许您对现有对象进行`set`和`delete`操作，同时可以更新实体本身和索引结构的存在过期时间。下面的例子显示了局部更新:

*Example 31. Simple Partial Update*

```java
public class SimplePartialUpateRunner implements ApplicationRunner {

    @Autowired
    private RedisKeyValueTemplate redisKeyValueTemplate;
    @Override
    public void run(ApplicationArguments args) throws Exception {
        PartialUpdate<Person> update = new PartialUpdate<>("16bbdf07...",Person.class)
                .set("firstname","Tom")								①     
                .set("address.city","Manhattan")					②
                .del("lastname");									③
        redisKeyValueTemplate.update(update);

        update = new PartialUpdate<>("16bbdf07...",Person.class)
                .set("address",new Address("America","NewYork"))	④
                .set("attributes", Collections.singletonMap("nickname","雅诗兰黛"));  ⑤
        redisKeyValueTemplate.update(update);

        update = new PartialUpdate<>("16bbdf07...",Person.class)
                .refreshTtl(true)									⑥
                .set("expiretion",60000L);
        redisKeyValueTemplate.update(update);
    }
}
```

① 将简单的firstname属性设置为Tom。

② 将简单的“address.city”属性设置为“Manhattan”，而不需要传入整个对象。

③ 删除lastname属性。

④ 设置复杂的address属性。

⑤ 设置一个map值，它删除以前存在的map，并用给定的值替换这些值。

⑥ 更改过期时间的同时，自动更新服务器过期时间。

> 更新复杂对象以及映射(或其他集合)结构需要与Redis进行进一步的交互，以确定现有的值，这意味着重写整个实体可能会更快。

### [8.10. Queries and Query Methods](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.queries)

### [8.11. Redis Repositories Running on a Cluster](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.cluster)

### [8.12. CDI Integration](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#redis.repositories.cdi-integration)

- 8.13. Redis Repositories Anatomy

  - [8.13.1. Insert new](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_insert_new)
  - [8.13.2. Replace existing](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_replace_existing)
  - [8.13.3. Save Geo Data](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_save_geo_data)
  - [8.13.4. Find using simple index](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_find_using_simple_index)
  - [8.13.5. Find using Geo Index](https://docs.spring.io/spring-data/redis/docs/2.1.3.RELEASE/reference/html/#_find_using_geo_index)
