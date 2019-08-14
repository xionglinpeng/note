# Java SPI

## SPI是什么

SPI全称Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的API，它可以用来启用框架扩展和替换组件。



Java SPI实际上是“基于接口的编程+策略模式+配置文件”组合实现的动态加载机制。

系统设计的各个抽象，往往有很多不同的实现方案，在面向对象的设计里，一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。为了实现在模块装配的时候可以不在程序里动态指明，这就需要一种服务发现机制。

Java SPI就是提供这样的一个机制：为某个接口寻找服务实现的机制。有点类似IOC的思想，就是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要。所以SPI的核心思想就是解耦。

## 使用场景

**调用者根据实际使用需要、启用、扩展、或者替换框架的实现策略**

案例

- 数据库驱动加载接口实现类加载

  配置数据库的时候常常会配置数据库驱动，例如mysql的驱动是`com.mysql.cj.jdbc.Driver`，这个驱动的加载方法就是通过Java SPI。打开mysql驱动jar包，可以发现，存在`META-INF/service`目录，里面有一个名为`java.sql.Driver`的文件，打开其文件，里面的内容就是配置驱动类。而其他数据库的驱动加载方式也是一样的。

- 日志门面接口实现类加载

- Spring

  Spring中大量使用了SPI，比如：对Servlet3.0规范对`javax.servlet.ServletContainerInitializer`接口的实现、自动类型装换Type conversion SPI（Converter SPI、Formatter SPI）等。

  Spring Boot具有类似机制，Spring Boot的自动装配是通过`SpringFactoriesLoader`类扫描所有jar包的`META-INF/spring.factories`文件代替SPI的`META-INF/service`目录下的描述文件，`SpringFactoriesLoader`就相当于JDK的`ServiceLoader`类

- Dubbo

  Dubbo中也大量使用SPI的方式实现框架的扩展，不过它对Java提供的原生SPI做了封装，允许用户扩展实现Filter接口。

## 使用介绍

1. 当服务提供者提供了接口的一种具体实后，在jar包的`META-INF/services`目录下创建一个以“接口全限定名”为命名的文件，内容为实现类的全限定名。
2. 接口实现类所在的jar包放在主程序的classpath下。
3. 主程序通过动态装载实现模块，它通过扫描目录下的配置文件找到实现类的全限定名，把类加载到JVM。
4. SPI的实现类必须有一个无参构造函数。

## 示例

```java
// 定义一个接口
package com.study.spi;
public interface RegisterSPI {
    String register();
}

//定义接口的实现：一
package com.study.spi;
public class FirstRegister implements RegisterSPI {
    @Override
    public String register() {
        return "Hello Register First.";
    }
}

//定义接口的实现：二
package com.study.spi;
public class SecondRegister implements RegisterSPI {
    @Override
    public String register() {
        return "Hello Second First.";
    }
}
```

创建服务提供接口文件，命名为`com.study.spi.RegisterSPI`（服务接口的全限定名），其文件必须在Classpath下的`META-INF/services`目录下

```yaml
src
	main
		resources
			META-INF
				services
					com.study.spi.RegisterSPI
```

服务提供接口文件的内容为其实现类的全限定名，多个实现类换行即可。

```
com.study.spi.FirstRegister
com.study.spi.SecondRegister
```

定义一个main方法，使用`ServiceLoader`来加载*服务提供接口文件*指定的实现。

```java
package com.study.spi;
import java.util.ServiceLoader;
public class SPIMain {
    public static void main(String[] args) {
        ServiceLoader<RegisterSPI> loader = ServiceLoader.load(RegisterSPI.class);
        for (RegisterSPI registerSPI : loader) {
            System.out.println(registerSPI.register());
        }
    }
}
```

测试输出：

```
Hello Register First.
Hello Second First.
```

## 原理分析



## 总结

- 优点

  使用Java SPI机制的优势是实现解耦，使得第三方服务模块的装配控制的逻辑与调用者的业务代码分离，而不是耦合在一个。应用程序可以根据实际业务情况启用框架扩展或替换框架组件。

- 缺点

  1. 虽然ServiceLoader也算是使用的延迟加载，但是基本只能通过遍历全部获取，也就是接口的实现类全部加载并实例化一遍。如果你并不想用某些实现类，它也被加载并实例化了，这就造成了浪费。获取某个实现类的方式不够灵活，只能通过Iterator形式获取，不能根据某个参数来获取对应的实现类。

  2. 多线程并发时，ServiceLoader类是线程不安全的。

