# Mybatis

## mybatis高级查询

### 高级结果映射

#### 一对一映射

通过一次查询将结果映射到不同对象的方式，称之为关联的前台结果映射。

关联的嵌套结果映射需要关联多个表将所有需要的值一次性查询出来。这种方式的好处是减少数据库查询次数，减轻数据库的压力，缺点是要写很复杂的SQL，并且当嵌套结果更复杂时，不容易一次写正确，由于要在应用服务器上将结果映射到不同的类上，因此也会增加应用服务器的压力。当一定会使用到嵌套结果，并且整个复杂的SQL执行速度很快时，建议使用关联嵌套结果查询。

6.1.1.2、使用`<resultMap>`配置一对一映射

6.1.1.3、使用`<resultMap>`的`<association>`标签配置一对一映射

`<association>`标签包含以下属性：

- `property`： 对应实体类中的属性名，必填项。
- `javaType`： 属性对应的java类型。
- `resultMap`： 可以直接使用现有的`resultMap`，而不需要在这里配置。
- `columnPrefix`： 查询列的前缀，配置前缀后，在子标签配置`<result>`的`column`时可以省略前缀。

6.1.1.4、`<association>`标签的嵌套查询

`<association>`标签的嵌套查询常用的属性如下：

- `select`：另一个映射查询的id，MyBatis会额外执行这个查询获取嵌套对应的结果。
- `column`：列名（或别名），将主查询中列的结果作为嵌套查询的参数，配置方式如`column={prop=col1,prop2=col2}`，`prop1`和`prop2`将作为嵌套查询的参数。
- `fetchType`：数据加载方式，可选值为`lazy`和`eager`，分别为延迟加载和积极加载，这个配置会覆盖全局的`lazyLoadingEnabled`配置。



特别提醒！

> ​	许多对延迟加载原理不太熟悉的朋友经常会遇到一些莫名其妙的问题：有些时候延迟加载可以得到数据，有些时候延迟加载就会报错，为什么会出现这种情况呢？
>
> ​	MyBatis延迟加载是通过动态代理实现的，当调用配置为延迟加载的属性方法时，动态代理的操作会被触发，这些额外的操作就是通过MyBatis的`SqlSession`去执行嵌套SQL的。由于在和某些框架集成时，`SqlSeesion`的生命周期交给了框架来管理，因此当对象超出`SqlSession`的生命周期调用时，会由于链接关闭等问题而抛出异常。在和Spring集成时，要确保只能在Service层调用延迟加载的属性。当结果从Service层返回至Controller层时，如果获取延迟加载的属性值，会因为`SqlSession`已经关闭而抛出异常。



#### 一对多映射



#### 鉴别器映射

`<discriminator>`标签常用的两个属性如下：

- `column`：该属性用于设置要进行鉴别比较值的列。
- `javaType`：该属性用于指定列的类型，保证使用相同的Java类型来比较值。

`<discriminator>`标签可以有1个或多个`<case>`标签，`<case>`标签包含以下三个属性：

- `value`：该值为`<discriminator>`指定`column`用来匹配的值。
- `resultMap`：当`column`的值和`value` 的值匹配时，可以配置使用`resultMap`指定的映射，`resultMap`优先级高于`resultType`。
- `resultType`：当`column`的值和`value`的值匹配时，用于配置使用`resultType`指定的映射。

`<case>`标签下面可以包含的标签和`resultMap`一样，用法也一样。

### 使用枚举或其他对象



## MyBatis缓存配置

### 一级缓存

​	MyBatis的一级缓存存在于`SqlSession`的生命周期中，在同一个SqlSession中查询时，MyBatis会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个Map对象中。如果同一个`SqlSession`中执行的方法和参数完全一致，那么通过算法会生成相同的键值。当Map缓存对象中已经存在该键值时，则会返回缓存中的对象。

如果不想让指定的select方法使用一级缓存，可以添加`flushCache`属性。

```xml
<select id=".." flushCache="true" resultMap="..">
    ...
</select>
```

这个属性配置为`true`后，会在查询数据前清空当前的一级缓存，因此该方法每次都会重新从数据库中查询数据。但是由于这个方法清空了一级缓存，影响当前`SqlSession`中所有缓存的查询，因此在需要反复查询获取只读数据的情况下，会增加数据库的查询次数，所以要避免这么使用。

`INSERT`、`UPDATE`、`DELETE`操作都会情况一级缓存。

### 二级缓存

#### 配置二级缓存

在MyBatis的全局配置`<settings>`中有一个参数`cacheEnabled`，这个参数是二级缓存的全局开关，默认值是`true`，初始状态为启用状态。如果把这个参数设置为`false`，即使有后面的二级缓存配置，也不会生效。由于这个参数值默认为`true`，所以不必配置，如果想要配置，可以在`mybatis-config.xml`中添加如下代码：

```xml
<settings>
	...
    <setting name="cacheEnabled" value="true"/>
    ...
</settings>
```

MyBatis的二级缓存是和命名空间绑定的，即二级缓存需要配置在`Mapper.xml`映射文件中，或者配置在`Mapper.java`接口中。在映射文件中，命名空间就是XML根节点`<mapper>`的`namespace`属性。在Mapper接口中，命名空间就是接口的全限定名称。

##### `Mapper.xml`中配置二级缓存



默认的二级缓存会有如下效果：

- 映射语句文件中的所有`SELECT`语句将会被缓存。
- 映射语句文件中的所有`INSERT`、`UPDATE`、`DELETE`语句会刷新缓存。
- 缓存会使用Least Recently Used（LRU，最近最少使用的）算法来回收。
- 根据时间表（如no Flush Interval，没有刷新间隔），缓存不会以任何时间顺序来刷新。
- 缓存会存储集合或对象（无论查询方法返回什么类型的值）的个引用。
- 缓存会被视为read/write（可以/可写）的，意味着对象检索不是共享的，而且可以安全地被调用者修改。而不干扰其他调用者或者线程所做的潜在修改。

所有的这些属性都可以通过缓存元素的属性来修改，如下：

```xml
<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>
```



`<cache>`配置属性：

- `eviction`：回收策略。
  - `LRU`（最近最少使用的）：移除最长时间不被使用的对象，这是默认值。
  - `FIFO`（先进先出）：按对象进入缓存的顺序来移除它们。
  - `SOFT`（软引用）：移除基于垃圾回收器状态和软引用规则的对象。
  - `WEAK`（若引用）：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
- `flushInterval`：（刷新间隔）。可以被设置为任意的正整数，而且它们代表一个合理的毫秒形式的时间段。默认情况不设置，即没有刷新间隔，缓存仅仅在调用语句时刷新。
- `size`：（引用数目）。可以被设置为任意正整数，要记住缓存的对象数目和运行环境的可用内存资源数目相关。默认值是1024。
- `readOnly`：（只读）。属性可以被设置为`true`或`false`。只读的缓存会给所有调用者返回缓存对象的相同实例，因此这些对象不能被修改，这提供了很重要的性能优势。可读写的缓存会通过序列化返回缓存对象的拷贝，这种方式会慢一下，但是安全，因此默认是false。

##### `Mapper`接口中配置二级缓存

在Mapper接口配置二级缓存，只需要在Mapper接口上添加`@CacheNamespace`注解即可。

```java
import org.apache.ibatis.annotations.CacheNamespace;
import org.apache.ibatis.cache.decorators.FifoCache;
@CacheNamespace
public interface Mapper {
}
```

它同样具有与XML配置一样的属性，示例如下：

```java
@CacheNamespace(eviction = FifoCache.class,flushInterval = 60000,size = 512,readWrite = true)
```

注意：

> 这里的`readWrite`属性与XML中的`readOnly`属性一样，用于配置缓存是否为只读类型，在这里`true`为读写，`false`为只读。默认为`true`。

注意：如果XML和Mapper接口同时开启了二级缓存会怎样？

> 如果同时开启了二级缓存，会抛出如下异常
>
> ```
> Caused by: java.lang.IllegalArgumentException: Caches collection already contains value for com.**.*Mapper
> ```
>
> 这是因为Mapper接口和对应的XML文件是相同的命名空间，想使用二级缓存，两者必须同时配置（如果接口不存在使用注解方式的方法，可以只在XML中配置），因此配置就会出错，这个时候应该使用参照缓存。

###### 参照缓存

Mapper接口配置参照缓存：

```java
@CacheNamespaceRef(*Mapper.class)
```

XML映射文件配置参照缓存：

```xml
<cache-ref namespace="com.**.*Mapper"/>
```



MyBatis中很少会同时使用Mapper接口注解方式和XML映射文件，所以参照缓存并不是为了解决这个问题而设计的。参照缓存除了能够通过引用其他缓存减少配置外，主要的作用是解决脏读。

#### 使用二级缓存

读写缓存：

​	MyBatis使用`SerializedCache(org.apache.ibatis.cache.decorators.SerializedCache)`序列化缓存类来实现可读写缓存，并通过序列化和反序列化来保证通过缓存获取数据时，得到的是一个新的实例。因此使用可读写缓存，可以使用`SerializedCache`序列化缓存。这个缓存类要求所有被序列化的对象必须实现`Serializable(java.io.Serializable)`接口。

只读缓存：

​	MyBatis就会使用`Map`来存储缓存值，这种情况下，从缓存中获取的对象就是同一个实例。



​	MyBatis默认提供的缓存实现是基于`Map`实现的内存缓存，已经可以满足基本的应用。但是当需要缓存大量的数据时，不能仅仅通过提高内存来使用MyBatis的二级缓存，还可以选择一些类似EhCache的缓存框架或Redis缓存数据库等工具来保存Mybatis的二级缓存数据。

#### 集成Redis缓存

[MyBatis Redis integration](http://www.mybatis.org/redis-cache/)

```xml
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-redis</artifactId>
    <version>1.0.0-beta2</version>
</dependency>
```

在`src/main/resources`目录下新增`redis.properties`配置文件

```xml
<cache type="org.mybatis.caches.redis.RedisCache"/>
```

其他集成：

- ignite-cache：https://github.com/mybatis/ignite-cache 
- couchbase-cache：https://github.com/mybatis/couchbase-cache 
- caffeine-cache：https://github.com/mybatis/caffeine-cache
- memcached-cache：https://github.com/mybatis/memcached-cache
- oscache-cache：https://github.com/mybatis/oscache-cache
- redis-cache：https://github.com/mybatis/redis-cache
- ehcache-cache：https://github.com/mybatis/ehcache-cache

#### 脏数据的产生和避免

​	二级缓存虽然能提高应用效率，减轻数据库服务器的压力，但是如果使用不当，很容易产生脏数据。这些脏数据会在不知不觉中影响业务逻辑，影响影响应用的实效，所以我们需要了解在MyBatis缓存中脏数据库是如何产生的，也要掌握避免脏数据的技巧。

​	MyBatis的二级缓存是和命名空间绑定的，所以通常情况下每一个Mapper映射文件都拥有自己的二级缓存，不同Mapper的二级缓存互不影响。

​	由于关系型数据库的设计，使得很多时候需要关联多个表才能获得想要的数据。在关联多表查询时坑爹会将该查询放到某个命名空间下的映射文件中，这样一个多表的查询就会缓存在该命名空间的二级缓存中。涉及这些表的增、删、改、操作通常不在一个映射文件中，它们的命名空间不同，因此当有数据变化时，多表查询的缓存未必会被清空，这种情况下就会产生脏数据库。

​	使用参照缓存，可以避免脏数据问题。当某几个表可以作为一个业务整体时，通常是让几个会关联的ER表同时使用同一个二级缓存，这样就能解决脏数据问题。

​	虽然这样可以解决脏数据的问题，但是并不是所有的关联查询都可以这么解决，如果有几十个表甚至所有表都以不同的关联关系存在于各自的映射文件中时，使用参照缓存显然没有意义。

#### 二级缓存适用场景

二级缓存虽然好处很多，但并不是什么时候都可以使用。以下场景中，推荐使用二级缓存：

1. 以查询为主的应用中，只有尽可能少的增、删、改操作。
2. 绝大多数以单表操作存在时，由于很少存在互相关联的情况，因此不会出现脏数据。
3. 可以按业务划分对表进行分组时，如关联的表比较少，可以通过参照缓存进行配置。

除了推荐使用的情况，如果脏读对系统没有影响，也可以考虑使用。在无法保证数据不出现脏读的情况下，建议在业务层使用可控制的缓存代替二级缓存。

##  `<settings>`配置：

- `cacheEnabled`：二级缓存的全局开关，默认值是`true`，初始状态为启用状态。

- `mapUnderscoreToCamelCase`：

- `aggressiveLazyLoading`：

- `lazyLoadTriggerMethods`：当调用配置中的方法时，加载全部的延迟加载数据。默认值为“`equals`”、“`clone`”、“`hashCode`”、“`toString`”。





## XML配置

### 类型处理（typeHandlers）

#### 处理枚举类型

























































































