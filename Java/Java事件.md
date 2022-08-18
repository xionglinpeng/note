# java事件

java的事件机制包括三部分：事件，事件监听器，事件源
在java.util包中提供了两个接口：`java.util.EventObject`和`java.util.EventListener`。

其源代码分别如下：

```java
package java.util;
/**
 * <p>派生所有事件状态对象的根类。<p>
 * 所有事件都是用一个"source"对象的引用构造，
 * 在逻辑上，这被认为是最初发生的事件的对象。
 * @since JDK1.1
 */
public class EventObject implements java.io.Serializable {
    private static final long serialVersionUID = 5516075349620653480L;
    protected transient Object  source;
    public EventObject(Object source) {
        if (source == null)
            throw new IllegalArgumentException("null source");
        this.source = source;
    }
    public Object getSource() {
        return source;
    }
    public String toString() {
        return getClass().getName() + "[source=" + source + "]";
    }
}
```

```java
package java.util;
/**
 * 一个所有事件监听器都必须扩展的标记接口。
 * @since JDK1.1
 */
public interface EventListener {
}
```

为了使在实际应用中事件监听的代码更具有通用性，所以以定义了如下抽象类ApplicationEvent和接口ApplicationListener。

```java
import java.util.EventObject;
public abstract class ApplicationEvent extends EventObject {
     private static final long serialVersionUID = 7099057708183571937L;
     public ApplicationEvent(Object source) {
           super(source);
     }
}
```

```java
import java.util.EventListener;
/**
 * EventListener接口本身是一个声明式的接口，没有任何抽象方法。
 * 为了是自定义监听器更具有通用性，所以定义了一个接口实现它，并且声明为泛型。
 * @param <E>
 */
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
     /**
      * 处理应用程序的事件。
      * @param 事件要响应
      */
     void onApplicationEvent(E event);
}
```

事件源

```java
import java.util.Collection;
import java.util.HashSet;
/**
 * 源对象，其作用是对监听器回调函数的调用。
 * 事件监听器只需要在程序启动的时候注册一次，所以源对象构造为一个单例对象。
 * @author xlp
 *
 */
public class Source {
     //监听器容器
     private Collection<ApplicationListener<?>> listeners;
     private static final Source SOURCE = new Source();
     //模拟对监听器的注册
     private Source() {
           listeners = new HashSet<>();
           listeners.add(new CustomListener1());
           listeners.add(new CustomListener2());
     }
     static Source getInstance(){
           return SOURCE;
     }
     /**
      * 调用监听器函数
      * @param event
      */
     public void publishEvent(ApplicationEvent event) {
           for (ApplicationListener<?> listener : listeners) {
                publishEvent(listener, event);
           }
     }
     /**
      * 因为泛型的原因，不能直接传递参数ApplicationEvent，所以使用函数二次包装。
      * @param listener
      * @param event
      */
     @SuppressWarnings({ "rawtypes", "unchecked" })
     public void publishEvent(ApplicationListener listener,ApplicationEvent event) {
           listener.onApplicationEvent(event);
     }
}
```

具体使用定义的自己的事件

```java
public class CustomEvent extends ApplicationEvent {
     private static final long serialVersionUID = 1297336406130680666L;
     private String name;
     public CustomEvent(Object source) {
           super(source);
     }
     public String getName() {
           return name;
     }
     public void setName(String name) {
           this.name = name;
     }  
}
```

监听器1
```java
public class CustomListener1 implements ApplicationListener<CustomEvent>{
     @Override
     public void onApplicationEvent(CustomEvent event) {
           System.out.println("my name is "+this.getClass().getName()+",The event that triggered me was "+event.getName());
     }
}
```

监听器2
```java
public class CustomListener2 implements ApplicationListener<CustomEvent> {
     @Override
     public void onApplicationEvent(CustomEvent event) {
           System.out.println("my name is "+this.getClass().getName()+",The event that triggered me was "+event.getName());
     }
}
```

测试
```java
public class Test {
     public static void main(String[] args) {
           //获取源对应
           Source source = Source.getInstance();
           //实例化想要触发的事件
           CustomEvent event = new CustomEvent(source);
           event.setName("join");
           //使用源对象触发事件
           source.publishEvent(event);
     }
}
```

输出结果
```
my name is com.java.demo.event.CustomListener1,The event that triggered me was join
my name is com.java.demo.event.CustomListener2,The event that triggered me was join
```

**原理**
在事件源对象中封装对监听器的注册，然后通过事件源对象的事件触发方法（publishEvent）调用监听器对象的回调方法（onApplicationEvent）。