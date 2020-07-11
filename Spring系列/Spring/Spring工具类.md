

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



## ResolvableType

# ResolvableType













## get

- `getType()`：

  获取其Type类型

- `getComponentType()`：

- `getRawClass()`：

- `getSource()`：

  获取原始对象，即当前`ResolvableType`包装的什么对象就返回什么对象。

- `getInterfaces()`：

  获取接口类型数组，每一个接口类型会被包装为`ResolvableType`类型。

- `getSuperType()`：

  获取父类类型，父类类型会被包装为`ResolvableType`类型。

- `getGenerics()`：

  获取参数化类型数组，每一个参数化类型会被包装为`ResolvableType`类型。

- `getGeneric(int... indexes)`：

  获取指定索引参数化类型，参数化类型会被包装为`ResolvableType`类型。

- `getNested(int nestingLevel)`：

- `getNested(int nestingLevel, Map<Integer, Integer> typeIndexesPerLevel)`：

## is

- `isArray()`：

  当前被包装的对象是否是数组类型。

- `isAssignableFrom(Class<?> other)`：

  判断此`ResolvableType`所包装的对象所表示的类或接口是否是指定Class参数所表示的类或接口的超类或超接口，或相等。如果是，则返回true，否则，返回false。

- `isAssignableFrom(ResolvableType other)`：

  判断此`ResolvableType`所包装的对象所表示的类或接口是否是指定`ResolvableType`参数包装的对象所表示的类或接口的超类或超接口，或相等。如果是，则返回true，否则，返回false。

- `isInstance(Object obj)`：

  判断此`ResolvableType`所包装的对象所表示的类或接口是否是指定Object参数所表示的类或接口的超类或超接口，或相等。如果是，则返回true，否则，返回false。

上述三个类型判断的方法其实现都是一样的，都会被转换为`ResolvableType`与`ResolvableType`之间的类型判定，即`isAssignableFrom(ResolvableType other)`方法的实现。

*类型判断示例：*

```java
ResolvableType resolvableType = ResolvableType.forClass(List.class);
System.out.println(resolvableType.isAssignableFrom(ArrayList.class)); //true
System.out.println(resolvableType.isAssignableFrom(ResolvableType.forClass(Collection.class))); //false
System.out.println(resolvableType.isInstance(new ArrayList<>())); //true
```

## as

- `asCollection()`：

  将此`ResolvableType`所包装的对象所表示的类或接口转换为表示`Collection.class`的`ResolvableType`对象。

- `asMap()`：

  将此`ResolvableType`所包装的对象所表示的类或接口转换为表示`Map.class`的`ResolvableType`对象。

- `as(Class<?> type)`：

  将此`ResolvableType`所包装的对象所表示的类或接口转换为表示指定`Class`参数类型的的`ResolvableType`对象。

*类型转换示例：*

```java
ResolvableType resolvableType = ResolvableType.forClass(List.class);
System.out.println(resolvableType.as(Collection.class)); // java.util.Collection<?>
System.out.println(resolvableType.asCollection()); // java.util.Collection<?>
System.out.println(resolvableType.asMap()); // ?
```

## has

- `hasGenerics()`：

  判断是否存在参数化类型。

- `hasUnresolvableGenerics()`：

  判断是否存在不可解析的泛型。只有以下两种情况下才会返回`true`：

  1. 通配符类型和类型变量。

     ```java
     public class TypeDemo<T> {
         private List<?> list; //通配符类型
         private List<T> d; //类型变量
         public static void main(String[] args) throws Exception {
             Field field = TypeDemo.class.getDeclaredField("list");
             ResolvableType resolvableType = ResolvableType.forField(field);
             System.out.println(resolvableType.hasUnresolvableGenerics()); //true
         }
     }
     ```

  2. 以原始的方式实现泛型接口，具体而言就是实现具有泛型接口，但未声明泛型。

     ```java
     //泛型接口
     public interface Interface<T> {
     }
     //以原始的方式实现泛型接口
     public class Impl implements Interface {
     }
     //测试
     public static void main(String[] args) throws Exception {
         ResolvableType resolvableType = ResolvableType.forClass(Impl.class);
         System.out.println(resolvableType.hasUnresolvableGenerics()); //true
     }
     ```

## resolve

- `resolve()`：

  解析此`ResolvableType`所包装对象所表示的类或接口的`Class`类型，如果不能解析，返回null。

- `resolve(Class<?> fallback)`：

  解析此`ResolvableType`所包装对象所表示的Type为Class对象，如果这个Type不能转换为Class对象，则返回指定参数的Class对象（`fallback`），否则，返回转换后的Class对象。

- `resolveGeneric(int... indexes)`：

  解析此`ResolvableType`所包装对象所表示的类或接口的指定索引的参数化类型的`Class`类型，如果不能解析，返回null。

- `resolveGenerics()`：

  解析此`ResolvableType`所包装对象所表示的类或接口的所有的参数化类型的`Class`类型，如果不能解析，返回null。

- `resolveGenerics(Class<?> fallback)`：

  解析此`ResolvableType`所包装对象所表示的类或接口的所有的参数化类型所表示的Type为Class对象，如果这个Type不能转换为Class对象，则返回指定参数的Class对象（`fallback`），否则，返回转换后的Class对象。它们以一个数组形式表示。

*类型解析示例：*

```java
public class TypeDemo<T> {
    //参数化类型
    private List<String> a;
    //通配符类型
    private List<? extends T> b;
}
```

类型可被转换：

```java
ResolvableType resolvableType = ResolvableType.forField(TypeDemo.class.getDeclaredField("a"));
System.out.println(resolvableType.getGeneric(0).resolve()); //class java.lang.String
System.out.println(resolvableType.getGeneric(0).resolve(Collection.class)); //class java.lang.String
System.out.println(resolvableType.resolveGeneric(0)); //class java.lang.String
System.out.println(resolvableType.resolveGenerics()[0]); //class java.lang.String
System.out.println(resolvableType.resolveGenerics(Collection.class)[0]); //class java.lang.String
```

类型不可被转换：

```java
ResolvableType resolvableType = ResolvableType.forField(TypeDemo.class.getDeclaredField("b"));
System.out.println(resolvableType.getGeneric(0).resolve()); //null
System.out.println(resolvableType.getGeneric(0).resolve(Collection.class)); //interface java.util.Collection
System.out.println(resolvableType.resolveGeneric(0)); //null
System.out.println(resolvableType.resolveGenerics()[0]); //null
System.out.println(resolvableType.resolveGenerics(Collection.class)[0]); //interface java.util.Collection
```

> 注意：`resolvableType.getGeneric(0).resolve()`等价于`resolvableType.resolveGeneric(0)`。

## GenericTypeResolver

### resolveTypeArgument

```java
public static Class<?> resolveTypeArgument(Class<?> clazz, Class<?> genericIfc)
```



## StopWatch

`org.springframework.util.StopWatch`



## StringDecoder

`org.springframework.core.codec.StringDecoder`