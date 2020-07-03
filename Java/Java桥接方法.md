# Java桥接方法



引用https://www.jianshu.com/p/2eaf4d5c168d

Java中的桥接方法是一种合成方法，在实现某些Java语言特性的时候是很有必要的。最为人熟知的例子就是**协变返回值类型**和**泛型擦除后导致的基类方法的参数与实际调用的方法参数类型不一致**以及**“改变”基类可见性**。

## 协变返回值类型

**泛型擦除**

看以下的例子

```java
public class Foo<T> {

    public T get(){
        return null;
    }
}

public class Bar extends Foo<string> {

    @Override
    public String get() {
        return null;
    }
}
```

如果你知道泛型擦除，那么就应该知道，泛型是在JDK1.5引入，它只在编译期起作用，所以上面的代码在泛型擦除之后，预期会变成这样：

```java
public class Foo {

    public Object get(){
        return null;
    }
}

public class Bar extends Foo {

    @Override
    public String get() {
        return null;
    }
}
```

简单的说，泛型T被擦除了，而在方法或字段中引用的泛型变成了`Object`。

看起来没有问题，也完全符合方法重写的规则，但是，我们将`Foo`和`Bar`使用`Javap -v`编译为字节码，内容如下：

*编译Foo.class*

```shell
# javap -v Foo.class
  ......
  public T get();
    descriptor: ()Ljava/lang/Object;
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aconst_null
         1: areturn
      LineNumberTable:
        line 6: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       2     0  this   Lcom/learn/java/bradgemetod/Foo;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       2     0  this   Lcom/learn/java/bradgemetod/Foo<TT;>;
    Signature: #16                          // ()TT;
}
Signature: #17                          // <T:Ljava/lang/Object;>Ljava/lang/Object;
SourceFile: "Foo.java" 
```

确实如我们所预期的那样，`T`变为了`java/lang/Object`。

*编译Bar.class*

```shell
# javap -v Bar.class
  ......
  public java.lang.String get();
    descriptor: ()Ljava/lang/String;
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aconst_null
         1: areturn
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       2     0  this   Lcom/learn/java/bradgemetod/Bar;

  public java.lang.Object get();
    descriptor: ()Ljava/lang/Object;
    flags: (0x1041) ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokevirtual #2                  // Method get:()Ljava/lang/String;
         4: areturn
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/learn/java/bradgemetod/Bar;
}
Signature: #16                          // Lcom/learn/java/bradgemetod/Foo<Ljava/lang/String;>;
SourceFile: "Bar.java"
```

看起来出乎意料，JVM合成了一个新的方法`public java.lang.Object get()`，这在源代码中是没有出现过的，这个新增的方法被标记为`ACC_BRIDGE`和`ACC_SYNTHETIC`，并且其转调了`public String get()`方法，它起了一个桥接的作用，编译器不得不这么做，因为在JVM方法中，返回类型也是方法签名的一部分，而桥接方法的创建就正好是实现协变返回值类型的方式。

> `ACC_BRIDGE`：这个方法是由编译生成的桥接方法。
>
> `ACC_SYNTHETIC`：这个方法是由编译器生成，并且不会在源代码中出现。

**桥接方法测试**

```java
public static void main(String[] args) throws NoSuchMethodException {
    Class<Bar> barClass = Bar.class;
    for (Method method : barClass.getDeclaredMethods()) {
        System.out.println(method.getReturnType().getName()
                           +" "+method.getName()
                           +"() is bridge method : "+method.isBridge());
    }
}

//控制台输出：
//java.lang.String get() is bridge method : false
//java.lang.Object get() is bridge method : true
```

## 泛型擦除入参类型不一致

看一下另一个代码示例，同样的编译源码：

```java
public class Foo<T> {

    public void set(T argument) {
        
    }
}

public class Bar extends Foo<String> {

    @Override
    public void set(String argument) {

    }
}
```

*编译Foo.class*

```shell
# javap -v Foo.class
  ......
  public void set(T);
    descriptor: (Ljava/lang/Object;)V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=2, args_size=2
         0: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   Lcom/learn/java/bradgemetod/Foo;
            0       1     1 argument   Ljava/lang/Object;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   Lcom/learn/java/bradgemetod/Foo<TT;>;
            0       1     1 argument   TT;
    Signature: #19                          // (TT;)V
}
Signature: #20                          // <T:Ljava/lang/Object;>Ljava/lang/Object;
SourceFile: "Foo.java"
```

*编译Bar.class*

```shell
# javap -v Bar.class
  ......
  public void set(java.lang.String);
    descriptor: (Ljava/lang/String;)V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=2, args_size=2
         0: return
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   Lcom/learn/java/bradgemetod/Bar;
            0       1     1 argument   Ljava/lang/String;

  public void set(java.lang.Object);
    descriptor: (Ljava/lang/Object;)V
    flags: (0x1041) ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: checkcast     #2                  // class java/lang/String
         5: invokevirtual #3                  // Method set:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/learn/java/bradgemetod/Bar;
}
Signature: #19                          // Lcom/learn/java/bradgemetod/Foo<Ljava/lang/String;>;
SourceFile: "Bar.java"
```

从上面的代码示例以及反编译结果看起来与*协变返回值类型*情况没有差别，这里的桥接方法覆盖了基类的`set`方法，不仅使用字符串参数将对自身的调用委派给字符串参数的`set`方法，还执行了一个到`java.lang.String`的类型转换检测（`#2`），这意味着，如果你执行如下代码，并忽略编译器的警告，将会从桥接方法哪里抛出`ClassCastException`异常。

```java
public static void main(String[] args) {
    Foo foo = new Bar();
    foo.set(new Object());
}
```

*输出结果：*

```java
Exception in thread "main" java.lang.ClassCastException: java.base/java.lang.Object cannot be cast to java.base/java.lang.String
	at com.learn.java.bradgemetod.Bar.set(Bar.java:3)
	at com.learn.java.bradgemetod.BradgeMethodTest.main(BradgeMethodTest.java:7)
```

**桥接方法测试**

```java
public static void main(String[] args) throws NoSuchMethodException {
    Class<Bar> barClass = Bar.class;
    Method setObject = barClass.getMethod("set", Object.class);
    System.out.println("set(Object) is bridge method : "+setObject.isBridge());
    Method setString = barClass.getMethod("set", String.class);
    System.out.println("set(String) is bridge method : "+setString.isBridge());
}

//控制台输出：
//set(Object) is bridge method : true
//set(String) is bridge method : false
```

## “改变”基类可见性

另外一种桥接方式是由于基类可见性问题引起的，参考如下示例：

编写Foo、Bar1、Bar2类，示例如下：

```java
class Foo {

    public void set(){

    }
}

public class Bar1 extends Foo {

    @Override
    public void set() {

    }
}

public class Bar2 extends Foo {

}
```

*编译Foo.class*

```shell
  public void set();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   Lcom/learn/java/bradgemetod/Foo;
}
SourceFile: "Foo.java"
```

*编译Bar1.class*

```java
  public void set();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   Lcom/learn/java/bradgemetod/Bar1;
}
SourceFile: "Bar1.java"
```

*编译Bar2.class*

```java
  public void set();
    descriptor: ()V
    flags: (0x1041) ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #2                  // Method com/learn/java/bradgemetod/Foo.set:()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/learn/java/bradgemetod/Bar2;
}
SourceFile: "Bar2.java"
```

`Foo`和`bar1`的字节码看起来平平无奇，跟源码一致，但是编译`bar2`，会发现其字节码中生成了桥接方法`set()`，并被标记为`ACC_BRIDGE`和`ACC_SYNTHETIC`。

编译器需要这样的方法，因为`Foo`类不是公开的，在`Foo`类所在包之外是不可见的，但是`bar2`类是公开的，它所继承来的所有方法在所在包之外都是可见的。需要注意的是，`bar1`类不会有桥接方法生成，因为它覆盖了`set`方法，因此没有必要“提升”其可见性。

虽然是生成了桥接方法，但是真的是因为`Foo`类不是公开的，在`Foo`类所在包之外是不可见的吗？为了验证这个问题，将`Foo`类修改为`public`，然后再次编译`bar2`类，编译后字节码如下：

```shell
  public com.learn.java.bradgemetod.Bar2();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method com/learn/java/bradgemetod/Foo."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/learn/java/bradgemetod/Bar2;
}
SourceFile: "Bar2.java"
```

可以看到，在`Foo`类为`public`的情况下，`bar2`并没有生成桥接方法。

**桥接方法测试**

```java
public static void main(String[] args) throws NoSuchMethodException {
        System.out.println("Bar1 set() is bridge method : "+
                           Bar1.class.getMethod("set").isBridge());
        System.out.println("Bar2 set() is bridge method : "+
                           Bar2.class.getMethod("set").isBridge());
}

//控制台输出：
//Bar1 set() is bridge method : false
//Bar2 set() is bridge method : true
```