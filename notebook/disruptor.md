# Disruptor



线程基础操作原语notify/wait系列，join方法，sleep方法，yeild方法

线程中断的理解

死锁的产生与避免

什么时候用户线程与deamon线程

什么是伪共享以及如何解决

Java内存模型是什么

什么是内存不可见性以及如何避免

volatile与Synchronized内存语义是什么，用来解决什么问题？

什么是指令重排序，如何避免？

什么是CAS操作，其出现为了解决什么问题，其本身存在什么问题、ABA问题是什么？

什么是原子性操作？

什么是独占锁，共享锁，公平锁，非公平锁？

Disruptor是英国外汇交易公司LMAX开发的一个高性能队列，研发的初衷是解决内存队列的延迟问题。与Kafka、RabbitMQ用于服务间的消息队列不同，disruptor一般用于线程间消息的传递。基于Disruptor开发的系统单线程能支撑每秒600万订单，2010年QCon演讲后，获得了业界关注。2011年，企业应用软件专家Martin Fowler专门撰写长文介绍The LMAX Architecture。同年它还或的了Oracle官方的Duke大奖。



对于`ArrayBlockingQueue`来说，最典型的应用场景就是生产者/消费者模式，所以，也按生产者/消费者模式来理解Disruptor进行学习。

先看一下代码的实践示例

**事件处理器**

这里的事件处理器等价于消费者，只是在Disruptor中事件处理器需要实现`EventHandler`接口，其语意为事件处理器。但是可以把它理解为消费者。

```java
import com.lmax.disruptor.EventHandler;
public class Consumer implements EventHandler<DataEvent> {
    @Override
    public void onEvent(DataEvent event, long sequence, boolean endOfBatch) throws Exception {
        System.out.println("I'm the consumer.");
    }
}
```



构建disruptor

生产者/消费者模式需要一个存储交互数据的地方，在使用`ArrayBlockingQueue`实现的时候，`ArrayBlockingQueue`就是存储交互的地方，而在Disruptor中，`RingBuffer`数据结构就是存储数据的地方。

```java
import com.lmax.disruptor.RingBuffer;
import com.lmax.disruptor.dsl.Disruptor;

public class DisruptorQueue {

    private final Disruptor<DataEvent> disruptor;

    public DisruptorQueue(){
        //构建Disruptor
        disruptor = new Disruptor<>(new DataEventFactory(),
                1024, r -> {
            Thread thread = new Thread(r,"setting name");
            //thread setting ...
            return thread;
        });
        //注册事件处理器
        disruptor.handleEventsWith(new DataEventHandler());
        //启动disruptor
        disruptor.start();
    }

    public RingBuffer<DataEvent> getRingBuffer(){
        return disruptor.getRingBuffer();
    }
}
```

发布事件

生产者/消费者模式中，生产者不需要特意构建，所有向生产者/消费者队列中发布数据的地方都是生产者。使用`ArrayBlockingQueue`实现时，数据是往`ArrayBlockingQueue`这个队列中发送数据，而Disruptor是通过事件承载数据，进而发布事件。

下面定义了一个main方法，和一个无限循环模拟生产者发布事件

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        DisruptorQueue queue = new DisruptorQueue();
        DataEventTranslatorOneArg oneArg = new DataEventTranslatorOneArg();
        for (int i = 0; true; i++) {
            queue.getRingBuffer().publishEvent(oneArg,
                                               new Random().nextInt(1000)+"");
            Thread.sleep(1000);
        }
    }
}
```



## 多消费者

多消费者的情况分为两类：

- 广播：对于多个消费者，每条信息会到达所有消费者，被多次处理，每个消费者业务逻辑一般不同，用于同一个消息的不同业务逻辑处理。
- 分组：对于同一组内多个消费者，每条消息只会被组内一个消费者处理，每个消费者业务逻辑一般相同，用于多消费者并发处理一组消息。

### 广播

- 消费者之间无依赖关系

  假设目前有handler1，handler2，handler3三个消费者处理同一批消息，每个消息都要被三个消费者处理到，三个消费者无依赖关系。

  ```java
  //注册事件处理器
  disruptor.handleEventsWith(handler1,handler2,handler3);
  ```

- 消费者之间有依赖关系

  假设handler3必须在handler1，handler2处理完成后进行处理。

  ```java
  //注册事件处理器
  disruptor.handleEventsWith(handler1,handler2).then(handler3);
  ```

### 分组

分组情况稍微不同，对于消费者，需要实现`WorkHandler<T>`而不是`EventHandler<T>`，接口定义分别如下：

```java
package com.lmax.disruptor;
public interface EventHandler<T> {
    void onEvent(T event, long sequence, boolean endOfBatch) throws Exception;
}
```

```java
package com.lmax.disruptor;
public interface WorkHandler<T> {
    void onEvent(T event) throws Exception;
}
```

注册分组类型的事件处理器：

```java
disruptor.handleEventsWithWorkerPool(handler1,handler12,handler13);
```