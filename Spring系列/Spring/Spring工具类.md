

## BeanUtils

### instantiateClass

```java
public static <T> T instantiateClass(Constructor<T> ctor, Object... args) throws BeanInstantiationException
```

## ConfigurationClassUtils



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

### isCacheSafe

## ValidationUtils



## PropertiesLoaderUtils

### loadProperties

```java
public static Properties loadProperties(Resource resource)
```



## LogFormatUtils





## WebAsyncUtils

`org.springframework.web.context.request.async.WebAsyncUtils`

### getAsyncManager(ServletRequest)

```java
public static WebAsyncManager getAsyncManager(ServletRequest servletRequest)
```

### getAsyncManager(WebRequest)

```java
public static WebAsyncManager getAsyncManager(WebRequest webRequest)
```

### createAsyncWebRequest(HttpServletRequest, HttpServletResponse)

```java
public static AsyncWebRequest createAsyncWebRequest(HttpServletRequest request, HttpServletResponse response)
```



## GenericTypeResolver

### resolveTypeArgument

```java
public static Class<?> resolveTypeArgument(Class<?> clazz, Class<?> genericIfc)
```



## StopWatch

`org.springframework.util.StopWatch`



