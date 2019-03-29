# ClassLoader

## 初始化

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