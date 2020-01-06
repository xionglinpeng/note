

## BeanUtils

### instantiateClass

```java
public static <T> T instantiateClass(Constructor<T> ctor, Object... args) throws BeanInstantiationException
```



## AopUtils

`org.springframework.aop.support.AopUtils`

## AopProxyUtils

`org.springframework.aop.framework.AopProxyUtils`

## ClassUtils

`org.springframework.util.ClassUtils`

### isPresent

```java
public static boolean isPresent(String className, @Nullable ClassLoader classLoader)
```

### forName

```java
public static Class<?> forName(String name, @Nullable ClassLoader classLoader)
```



## PropertiesLoaderUtils

### loadProperties

```java
public static Properties loadProperties(Resource resource)
```



## GenericTypeResolver

### resolveTypeArgument

```java
public static Class<?> resolveTypeArgument(Class<?> clazz, Class<?> genericIfc)
```



## StopWatch

`org.springframework.util.StopWatch`