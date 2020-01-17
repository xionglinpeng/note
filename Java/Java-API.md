git config --system --unset credential.helper

JAVA 6 API : <http://www.cjsdn.net/Doc/JDK60/>

JAVA 8 API : <https://docs.oracle.com/javase/8/docs/api/>

# AnnotatedElement

表示目前正在此VM中运行的程序的一个已注释的元素

## getAnnotation

```java
<T extends Annotation> T getAnnotation(Class<T> annotationClass)
```

如果存在该元素的指定类型的注释，则返回这些注释，否则返回null。

> 可获取继承注释。

```java
@Component
@SpringBootApplication
public class January{}

@Controller
public class February extends January{}
```

*Main方法测试：*

```java
public static void main(String[] args) {
    System.out.println("@Controller? : "+February.class.getAnnotation(Controller.class));
    System.out.println("@SpringBootApplication? : "+February.class.getAnnotation(SpringBootApplication.class));
    System.out.println("@Component? : "+February.class.getAnnotation(Component.class));
}
```

*结果输入如下：*

```
@Controller? : @org.springframework.stereotype.Controller(...)
@SpringBootApplication? : @org.springframework.boot.autoconfigure.SpringBootApplication(...)
@Component? : null
```

> `@SpringBootApplication`是可继承的注释。

## getAnnotations

```java
Annotation[] getAnnotations()
```

返回此元素上存在的所有注释。如果此元素没有注释，则返回长度为零的数组。该方法的调用者可以随意修改返回的数组，这不会对其他调用者返回的数组产生任何影响。

> 可获取继承注释。

```java
@Component
@SpringBootApplication
public class January{}

@Controller
public class February extends January{}
```

*Main方法测试：*

```java
public static void main(String[] args) {
	Arrays.stream(February.class.getAnnotations())
    	.forEach(System.out::println);
}
```

*结果输入如下：*

```
@org.springframework.boot.autoconfigure.SpringBootApplication(...)
@org.springframework.stereotype.Controller(...)
```

> `@SpringBootApplication`是可继承的注释。

## getAnnotationsByType

```java
default <T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass)
```

返回与此元素关联的注释，如果没有与此元素关联的注释，则返回长度为零的数组。此方法与getAnnotation(Class)的区别在于，此方法检测其参数是否是可重复的注释类型（JLS 9.6）,如果是，则尝试通过“looking through”容器注释来查找该类型的一个或多个注释。该方法的调用者可以随意修改返回的数组，这不会对其他调用者返回的数组产生任何影响。

> 可获取继承注释。

```java
@Controller
@SpringBootApplication
public class January{}

@PropertySource("")
@PropertySource("")
@Component
public class February extends January{}
```

*Main方法测试：*

```java
public static void main(String[] args) {
    Arrays.stream(February.class.getAnnotationsByType(PropertySource.class))
        .forEach(System.out::println);
    Arrays.stream(February.class.getAnnotationsByType(Component.class))
        .forEach(System.out::println);
    Arrays.stream(February.class
                 .getAnnotationsByType(SpringBootApplication.class))
        		 .forEach(System.out::println);
    Arrays.stream(February.class.getAnnotationsByType(Controller.class))
        .forEach(System.out::println);
}
```

*结果输入如下：*

```
@org.springframework.context.annotation.PropertySource(...)
@org.springframework.context.annotation.PropertySource(...)
@org.springframework.stereotype.Component(...)
@org.springframework.boot.autoconfigure.SpringBootApplication(...)
```

注意，上述结果没有输出`@Controller`注解。

> @PropertySource是可重复注释。
>
> `@SpringBootApplication`是可继承的注释。

## getDeclaredAnnotation

```java
default <T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass)
```

作用与`getAnnotation`相同，当前Declared方法与`getAnnotation`唯一的区别是：当前Declared方法将忽略继承的注释，而非`getAnnotation`方法不会忽略。

## getDeclaredAnnotations

```java
Annotation[] getDeclaredAnnotations()
```

作用与`getAnnotations`相同，当前Declared方法与`getAnnotations`唯一的区别是：当前Declared方法将忽略继承的注释，而`getAnnotations`方法不会忽略。

## getDeclaredAnnotationsByType

```java
default <T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass)
```

作用与`getAnnotationsByType`相同，当前Declared方法与`getAnnotationsByType`唯一的区别是：当前Declared方法将忽略继承的注释，而`getAnnotationsByType`方法不会忽略。

## isAnnotationPresent

**isAnnotationPresent**([Class](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)<? extends [Annotation](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Annotation.html)> annotationClass)

```java
default boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)
```

如果指定类型的注释存在于此元素上，则返回`true`，都返回`false`。此方法主要是为了便于访问标记注释而设计的。`isAnnotationPresent`不会忽略继承的注释。

示例：

```java
@Component
@SpringBootApplication
public class January{}

@Controller
public class February extends January{}
```

*Main方法测试：*

```java
public static void main(String[] args) {
    System.out.println("Has @Controller? : "+February.class.isAnnotationPresent(Controller.class));
    System.out.println("Has @SpringBootApplication? : "+February.class.isAnnotationPresent(SpringBootApplication.class));
    System.out.println("Has @Component? : "+February.class.isAnnotationPresent(Component.class));
}
```

*结果输入如下：*

```
Has @Controller? : true
Has @SpringBootApplication? : true
Has @Component? : false
```

> `@SpringBootApplication`是可继承的注释。

# AnnotatedType

`AnnotatedType`表示当前在次VM中运行的程序中对类型的潜在注释使用。可以使用Java编程语言中的任何类型，包括数组类型，参数化类型、类型变量或通配符类型。

`AnnotatedType`的源码如下，可以看到，`AnnotatedType`本身是一个接口，其直接继承了`AnnotatedElement`接口。

```java
public interface AnnotatedType extends AnnotatedElement {
    
    default AnnotatedType getAnnotatedOwnerType() {
        return null;
    }

    Type getType();
}
```

AnnotatedType自身提供了两个方法：

## getAnnotatedOwnerType

```java
default AnnotatedType getAnnotatedOwnerType()
```





## getType

```java
Type getType()
```

返回当前注释类型的基本类型。

# Class



```
public String toGenericString()
```





```
public String getTypeName()
```

返回此类型的名称的信息字符串。



## getAnnotatedSuperclass

```java
public AnnotatedType getAnnotatedSuperclass()
```

返回`AnnotatedType`对象，该对象表示Class对象的实体的超类。

如果这个Class对象没有显式地声明一个带注释的超类，那么返回的`AnnotatedType`对象表示一个没有注释的元素。

如果这个Class对象所表示的Object类是`interface`类型，`array`类型，`primitive`类型，或`void`，则返回`null`。

@Since: 1.8



```java
public class Monday{
}

public class Tuesday extends Monday{
}
```

```java
public static void main(String[] args) {
    AnnotatedType annotatedSuperclass = Tuesday.class.getAnnotatedSuperclass();
    System.out.println(annotatedSuperclass);
    System.out.println(annotatedSuperclass.getType());
}
```

结果输出如下：

```
sun.reflect.annotation.AnnotatedTypeFactory$AnnotatedTypeBaseImpl@6950e31
class com.jusdascm.test.ClassTest$Monday
```



## getAnnotatedInterfaces

```java
public AnnotatedType[] getAnnotatedInterfaces()
```



如果这个Class对象所表示的类或接口没有显式的声明任何带注释的超接口，则返回值为长度为0的数组。

如果这个Class对象所表示的Object类是`interface`类型，`array`类型，`primitive`类型，或`void`，则返回`null`。

@Since: 1.8



示例：

```java
public interface Day{
}
public interface Time{
}
public class Tuesday implements Day,Time{
}
```

*Main方法测试：*

```java
public static void main(String[] args) {
    AnnotatedType[] annotatedInterfaces = Tuesday.class.getAnnotatedInterfaces();
    for (AnnotatedType annotatedInterface : annotatedInterfaces) {
        System.out.println(annotatedInterface.getType());
    }
}
```

*结果输出如下：*

```java
interface com.jusdascm.test.ClassTest$Day
interface com.jusdascm.test.ClassTest$Time
```

## -isAnnotation

## -isAnnotationPresent

## -isAnonymousClass

## -isArray



## -isAssignableFrom

```java
public boolean isAssignableFrom(Class<?> cls)
```

Javadoc描述如下：

> 判定此Class对象所表示的类或接口与指定的Class参数所表示的类或接口是否相同，或是否是其超类或超接口。如果是则返回true；否则返回false。如果该Class表示一个基本类型，且指定的Class参数正是该Class对象，则该方法返回true；否则返回false。
>
> 特别：此方法能测试指定的Class参数所表示的类型能否转换为此Class对象所表示的类型。

例如`A.isAssignableFrom(B)`，简单的说就是`isAssignableFrom`方法可以判断A类是否是B类的超类或是B类的超接口，再或者是A类与B类相等。A类是调用方，B类是参数。

代码示例



## -isEnum

## -isInstance(Object obj)

## -isInterface()

## -isLocalClass()



## -isMemberClass

## isPrimitive

确定指定的类对象是否为基本类型。

有9个预定于Class对象，是8个基本类型和`void`。它们是由Java虚拟机创建，即`boolean`、`byte`、`char`、`short`、`int`、`long`、`float`、`double`。

这些对象只能通过以下公共静态`final`变量访问，并且是次方法返回`true`的唯一类对象。

`Boolean.TYPE`、`Byte.TYPE`、`Character.TYPE`、`Short.TYPE`、`Integer.TYPE`、`Long.TYPE`、`Float.TYPE`、`Double.TYPE`

示例：

```java
System.out.println("Short = "+Short.class.isPrimitive());
System.out.println("Integer = "+Integer.class.isPrimitive());
System.out.println("Long = "+Long.class.isPrimitive());
System.out.println("Boolean = "+Boolean.class.isPrimitive());
System.out.println("Byte = "+Byte.class.isPrimitive());
System.out.println("Character = "+Character.class.isPrimitive());
System.out.println("Float = "+Float.class.isPrimitive());
System.out.println("Double = "+Double.class.isPrimitive());
System.out.println("Double = "+Void.class.isPrimitive());
System.out.println("======================================");
System.out.println("Short = "+short.class.isPrimitive());
System.out.println("Integer = "+int.class.isPrimitive());
System.out.println("Long = "+long.class.isPrimitive());
System.out.println("Boolean = "+boolean.class.isPrimitive());
System.out.println("Byte = "+byte.class.isPrimitive());
System.out.println("Character = "+char.class.isPrimitive());
System.out.println("Float = "+float.class.isPrimitive());
System.out.println("Double = "+double.class.isPrimitive());
System.out.println("Double = "+void.class.isPrimitive());
```

*输出结果如下：*

```
Short = false
Integer = false
Long = false
Boolean = false
Byte = false
Character = false
Float = false
Double = false
Double = false
======================================
short = true
int = true
long = true
boolean = true
byte = true
char = true
float = true
double = true
void = true
```

**扩展**

有些情况下，在我们的业务操作中需要判断数据类型是否是基本类型，即使是基本类型的引用类型，也认为是基本类型，那么这种情况下`Class`对象的`isPrimitive()`就不能满足要求了，Spring框架提供了`ClassUtils`工具类，其中提供了`isPrimitiveWrapper`等方法，可以满足要求，示例如下：

```java
System.out.println("Short = "+ ClassUtils.isPrimitiveWrapper(Short.class));
System.out.println("Integer = "+ClassUtils.isPrimitiveWrapper(Integer.class));
System.out.println("Long = "+ClassUtils.isPrimitiveWrapper(Long.class));
System.out.println("Boolean = "+ClassUtils.isPrimitiveWrapper(Boolean.class));
System.out.println("Byte = "+ClassUtils.isPrimitiveWrapper(Byte.class));
System.out.println("Character = "+ClassUtils.isPrimitiveWrapper(Character.class));
System.out.println("Float = "+ClassUtils.isPrimitiveWrapper(Float.class));
System.out.println("Double = "+ClassUtils.isPrimitiveWrapper(Double.class));
```

*输出结果如下：*

```
ClassUtils Short = true
ClassUtils Integer = true
ClassUtils Long = true
ClassUtils Boolean = true
ClassUtils Byte = true
ClassUtils Character = true
ClassUtils Float = true
ClassUtils Double = true
```

## -isSynthetic

## -newInstance()

## -toGenericString()

## getComponentType

`public Class<?> getComponentType()`

返回表示数组组件类型的Class。如果此类不表示数组类，则此方法返回`null`。

示例：

```java
System.out.println(int[].class.getComponentType());
System.out.println(Long[].class.getComponentType());
System.out.println(Double[][].class.getComponentType());
System.out.println(String[].class.getComponentType());
System.out.println(Properties[].class.getComponentType());
System.out.println(String.class.getComponentType());
```

*输出结果如下：*

```
int
class java.lang.Long
class [Ljava.lang.Double;
class java.lang.String
class java.util.Properties
null
```

## getSigners

## getCanonicalName

返回java语言规范中所定义的底层类的规范化名称。如果底层类没有规范化名称（即如果底层类是一个组件类型没有规范化名称的本地类、匿名类或数组），则返回null。

示例：

```java
System.out.println(int.class.getCanonicalName());
System.out.println(Integer.class.getCanonicalName());
System.out.println(String.class.getCanonicalName());
System.out.println(Double[][].class.getCanonicalName());
System.out.println(Properties.class.getCanonicalName());
```

*输出结果如下：*

```
int
java.lang.Integer
java.lang.String
java.lang.Double[][]
java.util.Properties
```



## cast