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
| `@CacheEvict` | 表明Spring应该在缓存中清除一个或多个条目。                   |
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





## 源码分析

在开始源码分析之前，首先先说明一下Spring Cache源码的关键接口与类，以及它们之间的结构和关联关系。

核心类和接口如下：

- `org.springframework.cache.interceptor.BeanFactoryCacheOperationSourceAdvisor`
- `org.springframework.cache.interceptor.CacheInterceptor`

- `org.springframework.cache.interceptor.CacheOperationSource`
  - `org.springframework.cache.annotation.AnnotationCacheOperationSource`
  - `org.springframework.cache.interceptor.NameMatchCacheOperationSource`
  - `org.springframework.cache.interceptor.CompositeCacheOperationSource`
- `org.springframework.cache.interceptor.BasicOperation`
  - `org.springframework.cache.interceptor.CacheOperation`
    - `org.springframework.cache.interceptor.CacheEvictOperation`
    - `org.springframework.cache.interceptor.CachePutOperation`
    - `org.springframework.cache.interceptor.CacheableOperation`
- `org.springframework.cache.annotation.CacheAnnotationParser`
  - `org.springframework.cache.annotation.SpringCacheAnnotationParser`
  - `org.springframework.cache.transaction.TransactionAwareCacheDecorator`
- `org.springframework.cache.interceptor.CacheResolver`
  - `org.springframework.cache.interceptor.SimpleCacheResolver`
  - `org.springframework.cache.interceptor.NamedCacheResolver`
  - `org.springframework.cache.jcache.interceptor.SimpleExceptionCacheResolver`
  - `org.springframework.cache.jcache.interceptor.CacheResolverAdapter`
- `org.springframework.cache.CacheManager`
  - `org.springframework.cache.support.SimpleCacheManager`
  - `org.springframework.data.redis.cache.RedisCacheManager`
  - `org.springframework.cache.concurrent.ConcurrentMapCacheManager`
  - `org.springframework.cache.ehcache.EhCacheCacheManager`
  - `org.springframework.cache.jcache.JCacheCacheManager`
  - `org.springframework.cache.support.CompositeCacheManager`

- `org.springframework.cache.Cache`
  - `org.springframework.cache.concurrent.ConcurrentMapCache`
  - `org.springframework.cache.ehcache.EhCacheCache`
  - `org.springframework.data.redis.cache.RedisCache`
  - `org.springframework.cache.jcache.JCacheCache`
  - `org.springframework.cache.support.NoOpCache`
  - `org.springframework.cache.caffeine.CaffeineCache`

- `org.springframework.cache.interceptor.KeyGenerator`
  - `org.springframework.cache.interceptor.DefaultKeyGenerator`
  - `org.springframework.cache.jcache.interceptor.KeyGeneratorAdapter`
  - `org.springframework.cache.interceptor.SimpleKeyGenerator`

Spring Cache是基于AOP实现的，类`BeanFactoryCacheOperationSourceAdvisor`和`CacheInterceptor`就是Spring Cache AOP的切面和增强。

`CacheOperationSource`：缓存操作源，它封装了具有哪些缓存操作。

`CacheOperation`：缓存操作对象，对应具体的缓存操作，例如`@Cacheable`、`@CachePut`、`@CacheEvict`。

`CacheAnnotationParser`：缓存注解解析器，用于将具体的缓存操作注解解析为`CacheOperation`对象。

`CacheResolver`：缓存解析器，管理缓存管理器。

`CacheManager`：缓存管理器，管理具体的缓存，例如`RedisCache`、`EhCacheCache`、`JCacheCache`等。

`Cache`：缓存，实际操作缓存数据的对象，将数据写入缓存，或者从缓存删除，就是此对象完成这些具体工作。

`KeyGenerator`：键生成器，故名思义，就是缓存键的生成对象。

`BeanFactoryCacheOperationSourceAdvisor`在拦截方法的时候首先去缓存操作源（`CacheOperationSource`）中找到是否有对应的缓存操作`CacheOperation`，如果找到了就进行拦截。简单的说，就是被拦截的方法是否有标注缓存操作注解（`@Cacheable`、`@CachePut`、`@CacheEvict`），如果有标注，则进行拦截。而在缓存操作源中，就是通过缓存注解解析器（`CacheAnnotationParser`），将缓存操作注解转换为对应的缓存操作对象，然后返回。

一旦确认被拦截，就开始执行增强代码块，

我们从注解入口开始分析

org.springframework.cache.interceptor.CacheAspectSupport.CacheOperationContext

```java
@Nullable
	protected Object execute(CacheOperationInvoker invoker, Object target, Method method, Object[] args) {
		// Check whether aspect is enabled (to cope with cases where the AJ is pulled in automatically)
		if (this.initialized) {
			Class<?> targetClass = getTargetClass(target);
            //获取缓存操作源
			CacheOperationSource cacheOperationSource = getCacheOperationSource();
			if (cacheOperationSource != null) {
                //从缓存操作源中获取当前方法的缓存操作（缓存操作源中已经封装了有哪些缓存操作）
				Collection<CacheOperation> operations = cacheOperationSource.getCacheOperations(method, targetClass);
				if (!CollectionUtils.isEmpty(operations)) {
                    //将当前方法的缓存操作包装为CacheOperationContexts，并执行缓存操作
					return execute(invoker, method,
							new CacheOperationContexts(operations, method, args, target, targetClass));
				}
			}
		}

		return invoker.invoke();
	}
```





```java
@Nullable
private Object execute(final CacheOperationInvoker invoker, Method method, CacheOperationContexts contexts) {
   // 同步调用的特殊处理
   if (contexts.isSynchronized()) {
      CacheOperationContext context = contexts.get(CacheableOperation.class).iterator().next();
      if (isConditionPassing(context, CacheOperationExpressionEvaluator.NO_RESULT)) {
         Object key = generateKey(context, CacheOperationExpressionEvaluator.NO_RESULT);
         Cache cache = context.getCaches().iterator().next();
         try {
            return wrapCacheValue(method, cache.get(key, () -> unwrapReturnValue(invokeOperation(invoker))));
         }
         catch (Cache.ValueRetrievalException ex) {
            // 调用程序将所有可抛出的包装器包装在一个可抛出包装器实例中，
            // 这样我们就可以确保堆栈中有一个可抛出的包装器。
            // The invoker wraps any Throwable in a ThrowableWrapper instance so we
            // can just make sure that one bubbles up the stack.
            throw (CacheOperationInvoker.ThrowableWrapper) ex.getCause();
         }
      }
      else {
         // 不需要缓存，只需要调用底层方法
         return invokeOperation(invoker);
      }
   }

   // 处理前期所有的删除操作 
   // Process any early evictions
   processCacheEvicts(contexts.get(CacheEvictOperation.class), true,
         CacheOperationExpressionEvaluator.NO_RESULT);

   // 检查是否有匹配条件的缓存项，如果有缓存项，返回值包装器
   Cache.ValueWrapper cacheHit = findCachedItem(contexts.get(CacheableOperation.class));
   // 如果没有找到缓存项，使用缓存操作@Cacheable，将目标方法缓存值添加到缓存中
   List<CachePutRequest> cachePutRequests = new LinkedList<>();
   if (cacheHit == null) {
      collectPutRequests(contexts.get(CacheableOperation.class),
            CacheOperationExpressionEvaluator.NO_RESULT, cachePutRequests);
   }

   Object cacheValue;
   Object returnValue;

   if (cacheHit != null && !hasCachePut(contexts)) {
      // 如果没有@CachePuts缓存操作，就使用缓存命中
      cacheValue = cacheHit.get();
      returnValue = wrapCacheValue(method, cacheValue);
   }
   else {
      // 如果没有命中缓存，则调用目标方法
      returnValue = invokeOperation(invoker);
      cacheValue = unwrapReturnValue(returnValue);
   }

   // 使用缓存操作@CachePuts，将缓存值添加到缓存中
   collectPutRequests(contexts.get(CachePutOperation.class), cacheValue, cachePutRequests);

   // 处理从@Cacheable和@CachePut处收集的put请求
   for (CachePutRequest cachePutRequest : cachePutRequests) {
      cachePutRequest.apply(cacheValue);
   }
   // 处理后期所有的删除操作 
   processCacheEvicts(contexts.get(CacheEvictOperation.class), false, cacheValue);

   return returnValue;
}
```





```java
/**
  * 仅为传递的条件{@link CacheableOperation}查找缓存项。
  * @param contexts 缓存的操作
  * @return 保存缓存项的{@link Cache.ValueWrapper}，如果没有找到缓存项，则为{@code null}
  */
@Nullable
private Cache.ValueWrapper findCachedItem(Collection<CacheOperationContext> contexts) {
    Object result = CacheOperationExpressionEvaluator.NO_RESULT;
    for (CacheOperationContext context : contexts) {
        if (isConditionPassing(context, result)) {
            Object key = generateKey(context, result);
            Cache.ValueWrapper cached = findInCaches(context, key);
            if (cached != null) {
                return cached;
            }
            else {
                if (logger.isTraceEnabled()) {
                    logger.trace("No cache entry for key '" + key + "' in cache(s) " + context.getCacheNames());
                }
            }
        }
    }
    return null;
}
```



```java
@Nullable
private Cache.ValueWrapper findInCaches(CacheOperationContext context, Object key) {
    for (Cache cache : context.getCaches()) {
        Cache.ValueWrapper wrapper = doGet(cache, key);
        if (wrapper != null) {
            if (logger.isTraceEnabled()) {
                logger.trace("Cache entry for key '" + key + "' found in cache '" + cache.getName() + "'");
            }
            return wrapper;
        }
    }
    return null;
}
```



```java
public interface Cache {
    
   ......
       
   /**
    * 表示缓存值的(包装器)对象。
    */
    @FunctionalInterface
    interface ValueWrapper {
        /**
         * 返回缓存中的实际值。
         */
        @Nullable
        Object get();
    } 
    
    ......
        
}

```







```java
/**
 * Return the {@link CacheOperationMetadata} for the specified operation.
 * 返回指定操作的{@link CacheOperationMetadata}。
 * <p>Resolve the {@link CacheResolver} and the {@link KeyGenerator} to be
 * used for the operation.
 * <p>解析要用于操作的{@link CacheResolver}和{@link KeyGenerator}。
 * @param operation 操作
 * @param method 调用操作的方法
 * @param targetClass 目标类型
 * @return 已解析的操作元数据
 */
protected CacheOperationMetadata getCacheOperationMetadata(
      CacheOperation operation, Method method, Class<?> targetClass) {

   CacheOperationCacheKey cacheKey = new CacheOperationCacheKey(operation, method, targetClass);
   CacheOperationMetadata metadata = this.metadataCache.get(cacheKey);
   if (metadata == null) {
      KeyGenerator operationKeyGenerator;
      if (StringUtils.hasText(operation.getKeyGenerator())) {
         operationKeyGenerator = getBean(operation.getKeyGenerator(), KeyGenerator.class);
      }
      else {
         operationKeyGenerator = getKeyGenerator();
      }
      CacheResolver operationCacheResolver;
      if (StringUtils.hasText(operation.getCacheResolver())) {
         operationCacheResolver = getBean(operation.getCacheResolver(), CacheResolver.class);
      }
      else if (StringUtils.hasText(operation.getCacheManager())) {
         CacheManager cacheManager = getBean(operation.getCacheManager(), CacheManager.class);
         operationCacheResolver = new SimpleCacheResolver(cacheManager);
      }
      else {
         operationCacheResolver = getCacheResolver();
         Assert.state(operationCacheResolver != null, "No CacheResolver/CacheManager set");
      }
      metadata = new CacheOperationMetadata(operation, method, targetClass,
            operationKeyGenerator, operationCacheResolver);
      this.metadataCache.put(cacheKey, metadata);
   }
   return metadata;
}
```

