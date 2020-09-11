# ClassLoader



## class loader process

### 初始化

- `<clinit>()`方法由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问。例如

  ```java
  public class Test {
      static {
          i = 0;                 //给变量赋值可以正常编译通过
          System.out.println(i); //这句编译器会提示“Illegal forward reference”
      }
      static int i = 1;
  }
  ```

- `<clinit>()`方法与类的构造函数不同，它不需要显式地调用父类构造器，虚拟机会保证在子类`<clinit>()`方法执行之前，父类的`<clinit>()`方法以及执行完毕。因此在虚拟机中第一个被执行的`<clinit>()`方法的类肯定是`java.lang.Object`。

- 由于父类的`<clinit>()`方法先执行，也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作。例如：

  ```java
  static class Parent{
      public static int A = 1;
  
      static {
          A = 2;
      }
  }
  
  static class Sub extends Parent{
      public static int B = A;
  }
  
  public static void main(String[] args) {
      System.out.println(Sub.B); // 2
  }
  ```

- `<clinit>()`方法对于类或者接口来说并不是必需的，如果一个类中没有静态语句块，也就没有对变量的赋值操作，那么编译器可以不为这个类生成`<clinit>()`方法。



## class loader

- 启动类加载器（Bootstrap ClassLoader）：这个类加载器负责将存放在`<JAVA_HOME>\lib`目录中的，或者被`-Xbootclasspath`参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。
  启动类加载器无法被Java程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给引导类加载器，那直接使用null代替即可。
- 扩展类加载器（Extension ClassLoader）：这个加载器由`sum.misc.Launcher$ExtClassLoader`实现，它负责加载`<JAVA_HOME>\lib\ext`目录中，或者被`java.ext.dirs`系统变量所指定的路径中的所有类库，
  开发者可以直接使用扩展类加载器。
- 应用程序类加载器（Application ClassLoader）：这个类加载器由`sun.misc.Launcher$App-ClassLoader`实现。由于这个类加载器是ClassLoader中的`getSystemClassLoader()`方法返回值，所以一般也称它为系统类加载器。
  它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。



## 双亲委派模型

双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。这里类加载器之间的父子关系一般不会以继（Inheritance）承的关系来实现，而是都使用组合（Composition）关系来复用父加载器的代码。



双亲委派的工作过程：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。



双亲委派模型有一个显而易见的好处：java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如类`java.lang.Object`，它存放在rt.jar之中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。相关，如果没有使用双亲委派模型，由各个类加载器自行去加载的话，如果用户自己编写了一个称为`java.lang.Object`的类，并放在程序的ClassPath中，那系统中将会出现多个不同的Object类，Java类型提醒中最基础的行为也就无法保证，应用程序也将会变得一片混乱。



## java.lang.ClassLoader类介绍





## 自定义类加载器



### 文件系统类加载器



### 网络类加载器





## 线程上下文类加载器





## 扩展资料

- IBM Developer - [深入探讨 Java 类加载器](https://developer.ibm.com/zh/articles/j-lo-classloader/)

