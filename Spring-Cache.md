# Spring Cache

## 1、启用对缓存的支持

启用Spring对注解驱动缓存的支持：

bean配置

```java
@EnableCaching
```

xml配置

```xml
<cache:annotation-driven/>
```

其实在本质上，`@EnableCaching`和`<cache:annotation-driven/>`的工作方法是相同的。它们都会创建一个切面（aspect）并触发Spring缓存注解的切点（pointcut）。根据所使用的注解以及缓存的状态，这个切面会从缓存中获取数据，将数据添加到缓存之中或者从缓存中移出某个值。

缓存管理器是Spring缓存抽象的核心，它能够与多个流行的缓存实现进行集成。

### 1.1、配置缓存管理器

Spring 3.1内置了五个缓存管理器实现：

- `org.springframework.cache.support.SimpleCacheManager`
- `org.springframework.cache.support.NoOpCacheManager`
- `org.springframework.cache.concurrent.ConcurrentMapCacheManager`
- `org.springframework.cache.support.CompositeCacheManager`
- `org.springframework.cache.ehcache.EhCacheCacheManager`

其他缓存管理器：

- `org.springframework.data.redis.cache.RedisCacheManager`（来自于Spring Data Redis）
- `org.springframework.data.gemfire.cache.GemfireCacheManager`（来自于Spring Data GemFire）

例如：

```java
@Bean
public CacheManager cacheManager() {
    return new ConcurrentMapCacheManager();
}
```

### 1.2、使用Redis缓存

### 1.3、使用多个缓存管理器

我们并不是只能有且仅有一个缓存管理器。如果你很难确定该使用哪个缓存管理器，或者有合法的技术理由使用超过一个缓存管理器的话，那么可以尝试使用Spring的`CompositeCacheManager`。

`CompositeCacheManager`要通过一个或更多的缓存管理器来进行配置，它会迭代这些缓存管理器，以查找之前所缓存的值。

```java
@Bean
public CacheManager compositeCacheManager(RedisConnectionFactory connectionFactory) {
    CompositeCacheManager cacheManager = new CompositeCacheManager();
    List<CacheManager> cacheManagers = new ArrayList<>();
    cacheManagers.add(new ConcurrentMapCacheManager());
    cacheManagers.add(new SimpleCacheManager());
    cacheManagers.add(RedisCacheManager.create(connectionFactory));
    cacheManager.setCacheManagers(cacheManagers);
    return new CompositeCacheManager();
}
```

## 2、为方法添加注解以支持缓存

Spring的缓存抽象在很大程度上是围绕切面构建的。在Spring中启用缓存时，会创建一个切面，它会触发一个或更多的Spring的缓存注解。

**所有注解都能运用在方法或类上。当将其放在单个方法上时，注解所描述的缓存行为只会运用到这个方法上。如果注解放在类级别的话，那么缓存行为就会应用到这个类的所有方法上。**

Spring提供了四个注解来声明缓存规则：

| 注解          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| `@Cacheable`  | 表明Spring在调用方法之前，首先应该在缓存中查找方法的返回值。如果这个值能够找到，就会返回缓存的值。否则，这个方法就被被调用，返回值会放到缓存之中。 |
| `@CachePut`   | 表明Spring应该将方法的返回值放到缓存中。在方法的调用前并不会检查缓存，方法始终都会被调用。 |
| `@CacheEvict` | 表明Spring应该在缓存中清楚一个或多个条目。                   |
| `@Caching`    | 这时一个分组的注解，能够同时应用多个其他的缓存注解。         |

### 2.1、填充缓存

`@Cacheable`源码：

```java
package org.springframework.cache.annotation;
......
/**
 * @since 3.1
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Cacheable {

   @AliasFor("cacheNames")
   String[] value() default {};

   @AliasFor("value")
   String[] cacheNames() default {};

   String key() default "";

   String keyGenerator() default "";

   String cacheManager() default "";

   String cacheResolver() default "";

   String condition() default "";

   String unless() default "";

   boolean sync() default false;
}
```

`@CachePut`源码

```java
package org.springframework.cache.annotation;
......
/**
 * @since 3.1
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CachePut {

   @AliasFor("cacheNames")
   String[] value() default {};

   @AliasFor("value")
   String[] cacheNames() default {};

   String key() default "";

   String keyGenerator() default "";

   String cacheManager() default "";

   String cacheResolver() default "";

   String condition() default "";

   String unless() default "";
}
```

`@Cacheable`和`@CachePut`有一些属性是共有的。

| 属性        | 类型     | 描述                                                         |
| ----------- | -------- | ------------------------------------------------------------ |
| `value`     | String[] | 要使用的缓存名称。                                           |
| `condition` | String   | SpEL表达式，如果得到的值是false的话，不会将缓存应用到方法调用上。 |
| `key`       | String   | SpEL表达式，用来计算自定义的缓存key。                        |
| `unless`    | String   | SpEl表达式，如果得到的值是true的话，返回值不会方法缓存之中。 |

example:

```java
@Override
@Cacheable(cacheNames = "city",key = "'cities'")
public List<City> getAllCities() {
    ......
}
@Override
@CachePut(cacheNames = "city",key = "'cities'")
public BaseEntity saveCity(CityDto cityDto) {
    ......
}
```

**注解可以放在接口的抽象方法上，其接口的所有实现类都会应用相同的缓存规则。**

```java
@Cacheable(cacheNames = "city",key = "'cities'")
List<City> getAllCities();
```

### 2.2、自定义缓存key

缓存key：默认的缓存key要基于方法的参数来确定。

`@Cacheable`和`@CachePut`都有一个名为`key`的属性，这个属性能够替换默认的key，它是通过一个SpEL表达式计算得到的。任意的SpEL表达式都是可行的，但是更常见的场景是所定义的表达式与存储的缓存中的值有关，据此计算得到key。

表：Spring提供了多个用来定义缓存规则的SpEL扩展

| 表达式              | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| `#root.args`        | 传递给缓存方法的参数，形式为数组。                           |
| `#root.caches`      | 该方法执行时所对应的缓存，形式为数组。                       |
| `#root.target`      | 目标对象。                                                   |
| `#root.targetClass` | 目标对象的类，是`#root.target.class`的简写形式。             |
| `#root.method`      | 缓存方法                                                     |
| `#root.methodName`  | 缓存方法的名字，是`#root.method.name`的简写形式。            |
| `#result`           | 方法调用的返回值（不要用在`@Cacheable`注解上）               |
| `#Argument`         | 任意的方法参数名（如`#argName`）或参数索引（如`#a0`或`#p0`） |

example:

```java
@Override
@CachePut(cacheNames = "city",key = "#result.id")
public BaseEntity saveCity(CityDto cityDto) {
    ......
}
```

### 2.3、条件化缓存

- `unless`：控制返回的数据是否放置在缓存中，但是仍然获取缓存中查找。
  - `true`->不放在缓存中，`false`->放在缓存中；

- `condition`：控制是否从缓存中查找数据。
  - `true`->从缓存中查找，`false`->不从缓存中查找，返回值也不会放在缓存中；

`unless` example：

参数name如果等于“ABC”，则返回的城市对象不放在缓存中

```java
@Cacheable(cacheNames = "city",unless = "#result.name.equals('ABC')")
public City getCityByName(String name){
    ......
}
```

`condition` example:

参数name如果等于“ABC”，则从缓存中查询对象

```java
@Cacheable(cacheNames = "city",condition = "#root.args[0].equal('ABC')")
public City getCityByName(String name){
    ......
}
```

**注意：**

> 因为`unless`是控制返回的数据是否放置在缓存中，所以其可以使用`#result`引用返回值。但是`condition`是控制是否从缓存中查找数据，所以它不能等到方法返回时再确定是否该关闭缓存，这意味着它的表达式必须要在进入方法时进行计算，所以我们不能通过`#result`引用返回值。

### 2.4、 移出缓存条目

`@CacheEvict`用于移出缓存条目。

```java
@Override
@CacheEvict(cacheNames = "city",key = "'all_cities'")
public Boolean removeCity(String uuid) {
	......
}
```

**注意：**与`@Cacheable`和`@CachePut`不同，`@CacheEvict`能够应用在返回值为`void`的方法上，而`@Cacheable`和`@CachePut`需要非`void`的返回值，它将会作为放在缓存中的条目。因为`@CacheEvict`只是将条目从缓存中移除，因此它可以放在任意的方法上，甚至`void`方法。

`@CacheEvict`源码：

```java
package org.springframework.cache.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.core.annotation.AliasFor;

/**
 * @since 3.1
 * @see CacheConfig
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CacheEvict {

	@AliasFor("cacheNames")
	String[] value() default {};

	@AliasFor("value")
	String[] cacheNames() default {};

	String key() default "";

	String keyGenerator() default "";

	String cacheManager() default "";

	String cacheResolver() default "";

	String condition() default "";

	boolean allEntries() default false;

	boolean beforeInvocation() default false;
}
```

`@CacheEvict`注解的属性：

| 属性                | 类型       | 描述                                                         |
| ------------------- | ---------- | ------------------------------------------------------------ |
| `value[cacheNames]` | `String[]` | 要使用的缓存名称。                                           |
| `key`               | `String`   | SpEL表达式，用来计算自定义的缓存key。                        |
| `keyGenerator`      | `String`   |                                                              |
| `cacheManager`      | `String`   |                                                              |
| `cacheResolver`     | `String`   |                                                              |
| `condition`         | `String`   | SpEL表达式，如果得到的值是`true`的话，缓存不会应用到方法调用上。 |
| `allEntries`        | `boolean`  | 如果为`true`的话，特定缓存的所有条目都会被移出掉。           |
| `beforeInvocation`  | `boolean`  | 如果为`true`的话，在方法调用之前移出条目。如果为`false`（默认值）的话，在方法成功调用之后再移出条目。 |

### 2.5、应用多个缓存注解

`@Caching`源码：

```java
package org.springframework.cache.annotation;
/**
 * @since 3.1
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Caching {

   Cacheable[] cacheable() default {};

   CachePut[] put() default {};

   CacheEvict[] evict() default {};
}
```

## 3、使用XML声明缓存

为什么想要以XML的方法声明缓存？如果我们需要再没有源码的bean上应用缓存功能，那么以XML的配置方式就很有效了。

Spring的cache命名空间提供了使用XML声明缓存规则的方法，可以作为面向注解缓存的替代方案。因为缓存是一种面向切面的行为，所以cache命名空间会与Spring的aop命令空间结合起来使用，用来声明缓存所应用的切点在哪里。

要开始配置XML声明的缓存，首先需要创建Spring配置文件，这个文件中要包含cache和aop命名空间：

















