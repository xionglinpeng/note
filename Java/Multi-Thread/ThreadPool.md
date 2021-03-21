# 线程池





## 线程池原理

线程池提供了核心线程数/最大线程数/线程的空闲时间/任务等待队列/拒绝策略，它们之间的关系是当向线程池添加任务时，如果线程池的线程数小于核心线程数，则会创建一个新的线程来执行任务（无论核心线程是否空闲）；如果线程池中的线程数大于等于核心线程数，则会将新任务添加到任务等待队列；如果任务等待队列也满了，则在总线程数不大于最大线程数的情况下创建新线程执行任务（有空闲线程时使用空闲线程）；若已达到最大线程数了，则执行饱和（也可以叫拒绝）策略（在队列满了，线程数也满了的时候执行的逻辑）。



## ThreadPoolExecutor

`ThreadPoolExecutor`类有四个构造函数，如下：

```java
ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,
                   TimeUnit unit,BlockingQueue<Runnable> workQueue)
    
ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,
                   TimeUnit unit,BlockingQueue<Runnable> workQueue,
                   ThreadFactory threadFactory)
    
ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,
                   TimeUnit unit,BlockingQueue<Runnable> workQueue,
                   RejectedExecutionHandler handler)
    
ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,
                   TimeUnit unit,BlockingQueue<Runnable> workQueue,
                   ThreadFactory threadFactory,RejectedExecutionHandler handler)
```

总共有7个参数：

- `int corePoolSize`：线程池的核心线程数。
- `int maximumPoolSize`：线程池的最大线程数。
- `long keepAliveTime`：线程池中空闲线程的存活时间。
- `TimeUnit unit`：keepAliveTime的单位。
- `BlockingQueue<Runnable> workQueue`：用于存放等待执行任务的阻塞队列。
- `ThreadFactory threadFactory`：线程池创建线程的工厂。
- `RejectedExecutionHandler handler`：饱和策略。



## 任务等待队列

它是一个`BlockingQueue`对象，有如下几种选择：

- 直接提交队列：该功能由`SynchronousQueue`队列提供。`SynchronousQueue`是一个特殊的阻塞队列，它没有容量，每一个插入操作都要等一个相应的删除操作，反之，删除也要等待相应的插入。所以，当线程池使用`SynchronousQueue`队列时，提交的任务不会被真实的保存，而总是将新任务提交给线程池执行，如果没有空闲的线程，则尝试创建新的线程执行，如果已达到线程池的最大线程数，则执行拒绝策略。因此，在使用`SynchronousQueue`队列时，通常需要设置较大的`maximumPoolSize`值，否则很容易执行拒绝策略。

- 有界任务队列：可以通过`ArrayBlockingQueue`队列实现。完全遵循线程池的实现原理逻辑。

- 无界任务队列：可以通过`LinkedBlockingQueue`队列实现。与有界任务队列相比，除非系统资源耗尽，否则无界任务队列不存在入队失败的情况。若任务创建和处理速度差异很大，无界队列会保持快速增长，直到耗尽系统内存。

- 优先任务队列：通过`PriorityBlockingQueue`实现。它是具有执行优先级的队列，可以控制任务的执行先后顺序

## 饱和策略

线程池的饱和策略是指在队列，线程数都满了的时候执行的逻辑策略，

`ThreadPoolExecutor`默认提供了四种饱和策略：

- `AbortPolicy`：该策略会直接抛出异常，阻止系统正常工作。此策略是默认策略。

- `CallerRunsPolicy`：该策略会使用调用者线程执行当前任务

- `DiscardOldestPolicy`：该策略会丢弃队列里最近的一个任务，并执行当前任务。

- `DiscardPolicy`：该策略会直接丢弃任务。

以上策略如果无法满足要求，还可以自定义扩展饱和策略，只需要实现`RejectedExecutionHandler`接口即可，源码如下：

```java
package java.util.concurrent;
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

`r`表示请求执行的任务，`executor`是当前线程池。

## 自定义线程创建

线程池的线程创建是通过`ThreadFactory`来创建的，`ThreadFactory`是一个接口，源码如下：

```java
public interface ThreadFactory {
    Thread newThread(Runnable r);
}
```

`Executors`类默认提供了一个线程工厂，而`ThreadPoolExecutor`类的构造中就是默认使用此线程工厂。

源码如下：

```java
public class Executors {
    ......
    public static ThreadFactory defaultThreadFactory() {
    	return new DefaultThreadFactory();
    }
    ......
    private static class DefaultThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        DefaultThreadFactory() {
            SecurityManager s = System.getSecurityManager();
            group = (s != null) ? s.getThreadGroup() :
            Thread.currentThread().getThreadGroup();
            namePrefix = "pool-" +
                poolNumber.getAndIncrement() +
                "-thread-";
        }

        public Thread newThread(Runnable r) {
            Thread t = new Thread(group, r,
                                  namePrefix + threadNumber.getAndIncrement(),
                                  0);
            if (t.isDaemon())
                t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY)
                t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    }
    ......
}
```

我们可以定义自己的线程工厂实现，自定义线程工厂可以帮助我们做不少事，比如可以跟踪线程池究竟在何时创建了多少线程，也可以自定义线程的名称/组以及优先级等信息，甚至可以任性地将所有线程设置为守护线程。总之，使用自定义线程池可以让我们更加自由的设置池子中所有线程的状态。

> 使用开源框架Guava提供的`ThreadFactoryBuilder`可以快速的给线程池的线程设置有意义的信息。代码如下：
>
> ```java
> ThreadFactoryBuilder builder = new ThreadFactoryBuilder();
> ThreadFactory threadFactory = builder.setDaemon(true).setNameFormat("my-thread-%s").build();
> ```

## 线程池的创建

线程池的创建通过`ThreadPoolExecutor`类创建，前面已经说过，它有四个重载的构造函数，并且也对所有构造参数进行了说明，需要注意的是`ThreadFactory`和`RejectedExecutionHandler`可以不用设置，`ThreadPoolExecutor`提供了默认的实现，查看源码，`ThreadPoolExecutor`使用`AbortPolicy`作为默认的饱和策略，如下：

```java
private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();
```

而线程工厂`Executors`类中提供的默认线程工厂：`Executors.defaultThreadFactory()`。

## 关闭线程池

线程池提供了两个关闭方法：

- `executor.shutdown();`不会立即终止线程池，而是等待所有缓存在队列中的任务以及正在执行的任务都执行完毕，才会终止。等待期间不会接受新的任务。
- `executor.shutdownNow();`立即终止线程池，并尝试中断正在执行的任务，并清空所有缓存在队列中的任务，返回尚未执行的任务。

为了验证线程池的关闭确实如上所述，定义了一个线程池进行验证，此线程池的核心线程数和最大线程数都是2，任务队列为`ArrayBlockingQueue`，容量为3。每个任务休眠2秒模拟任务的执行。

*验证shutdown：*

```java
public static void main(String[] args) {
    ThreadPoolExecutor executor = new ThreadPoolExecutor(2,2,60, 			       
                             TimeUnit.MICROSECONDS,new ArrayBlockingQueue<>(3));
    for (int i = 0; i < 5; i++) {
        if (i == 4) executor.shutdown();
        try {
            executor.execute(()->{
                System.out.println(Thread.currentThread().getName() + " execute");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    System.out.println(Thread.currentThread().getName() + 
                                       "被中断了");
                    return;
                }
                System.out.println(Thread.currentThread().getName() + 
                                   " execute finish.");
            });
        } catch (RejectedExecutionException e) {
            System.out.println("拒绝执行任务");
        }
    }
    System.out.println("shutdown = " + executor.isShutdown());
    System.out.println("terminated = "+ executor.isTerminated());
}
```

输出结果如下：

```
pool-1-thread-1 execute
pool-1-thread-2 execute
拒绝执行任务
shutdown = true
terminated = false
pool-1-thread-1 execute finish.
pool-1-thread-1 execute
pool-1-thread-2 execute finish.
pool-1-thread-2 execute
pool-1-thread-1 execute finish.
pool-1-thread-2 execute finish.
```

从输出的结果可以看出`shutdown`的执行是在所有任务执行完之前完成。输出结果中输出了4次`finish`，通过也答应出了”拒绝执行任务“，可以证明确认`shutdown`确实不会立即终止线程池，而是等待所有缓存在队列中的任务以及正在执行的任务都执行完毕，才会终止。等待期间不会接受新的任务。

没有输出”被中断了“，也证明了`shutdown`不会中断线程。

*验证shutdownNow：*

```java
public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(2,2,
                60, TimeUnit.MICROSECONDS,new ArrayBlockingQueue<>(3));
        for (int i = 0; i < 5; i++) {
            if (i == 4) {
                List<Runnable> runnables = executor.shutdownNow();
                for (int j = 0; j < runnables.size(); j++) {
                    System.out.println(
                        "thread"+(j+1)+" : "+ runnables.get(j).hashCode());
                }
            }
            try {
                executor.execute(()->{
                    System.out.println(
                        Thread.currentThread().getName() + " execute");
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        System.out.println(
                            Thread.currentThread().getName() + "被中断了");
                        return;
                    }
                    System.out.println(
                        Thread.currentThread().getName() + " execute finish.");
                });
            } catch (RejectedExecutionException e) {
                System.out.println("拒绝执行任务");
            }

        }
        System.out.println("shutdown = " + executor.isShutdown());
        System.out.println("terminated = "+ executor.isTerminated());
    }
```

输出结果 如下：

```
pool-1-thread-1 execute
pool-1-thread-2 execute
thread1 : 1175962212
thread2 : 1175962212
pool-1-thread-1被中断了
pool-1-thread-2被中断了
拒绝执行任务
shutdown = true
terminated = true
```

从输出结果可以证明`shutdownNow`中断了线程，清空并返回了任务队列中的线程。至于拒绝接受新任务，那也是应有之意。

## 扩展线程池

`ThreadPoolExecutor`线程池在每个线程的每次执行前后，以及关闭线程池的时候提供了一些钩子方法，如下：

```java
protected void beforeExecute(Thread t, Runnable r) { }
protected void afterExecute(Runnable r, Throwable t) { }
protected void terminated() { }
```

通过继承`ThreadPoolExecutor`类，并重写这些钩子方法，可以自定义一些增强功能。

例如，记录每个线程每次执行的时长：

定义了一个`CustomThreadPoolExecutor`类，它继承了`ThreadPoolExecutor`类，并重写了钩子方法。

```java
public class CustomThreadPoolExecutor extends ThreadPoolExecutor {

    private static final ThreadLocal<Long> threadLocal = new ThreadLocal<>();

    public CustomThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    @Override
    protected void beforeExecute(Thread thread, Runnable runnable) {
        System.out.println(Thread.currentThread().getName() + 
                           " before execute ...");
        threadLocal.set(System.currentTimeMillis());
    }

    @Override
    protected void afterExecute(Runnable runnable, Throwable throwable) {
        System.out.println(Thread.currentThread().getName() + 
                           " after execute ...");
        System.out.println(Thread.currentThread().getName() + 
                           " execute time : "+(System.currentTimeMillis() - threadLocal.get()));
        threadLocal.remove();
    }

    @Override
    protected void terminated() {
        System.out.println(Thread.currentThread().getName() + " terminated ...");
    }
}
```

Main方法测试：

```java
public static void main(String[] args) {
    ThreadPoolExecutor executor = new CustomThreadPoolExecutor(5,5,60, TimeUnit.MICROSECONDS,new ArrayBlockingQueue<>(10));
    for (int i = 0; i < 2; i++) {
        executor.execute(()->{
            System.out.println(Thread.currentThread().getName() + " execute");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
    executor.shutdown();
}
```

输出结果：

```
pool-1-thread-1 before execute ...
pool-1-thread-2 before execute ...
pool-1-thread-2 execute
pool-1-thread-1 execute
pool-1-thread-1 after execute ...
pool-1-thread-2 after execute ...
pool-1-thread-1 execute time : 2002
pool-1-thread-2 execute time : 2001
pool-1-thread-1 terminated ...
```

可以看到虽然多个线程之间是乱序的，但是在单个线程之间，钩子方法的执行顺序始终是

```
beforeExecute() —> run() —> afterExecute() —> terminated()
```

## 合理配置线程池

线程池的大小并不是越大越好，需要根据具体的情况合理的配置大小，可以从以下几个角度去考虑：

- 任务的性质：CPU密集型任务/IO密集型任务/混合型任务
- 任务的优先级：高/中/低
- 任务的执行时间：长/中/短
- 任务的依赖性：是否依赖其他系统资源，如数据库连接。

CPU密集型任务应配置尽可能小的线程数，一般配置N~cpu~-1个线程数。

IO密集型任务应配置尽可能多的线程数，一般配置2*N~cpu~个线程数。

混合型任务，如果可以拆分，可以将其拆分成一个CPU密集型任务和一个IO密集型任务，如果两个任务的执行时间相差不大，那么分解后执行的吞吐量将高于串行执行的吞吐量。如果两个任务的执行时间相差太大，则没必要进行分解。

优先级不同的任务可以使用优先级队列`PriorityBlockingQueue`来处理。它可以让优先级高的任务先执行（考虑到物理机对线程优先级的支持，从优先级的角度考虑效果可能不会太好）。

执行时间不同的任务可以交给不同规模的线程池来处理，或者可以使用优先级队列，让执行时间短的任务先执行。

依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，等待的时候越长，则CPU空闲时间就越长，那么线程数应该设置的越大，这样才能更好的利用CPU（其实这个就是IO密集型任务）。



一般来说，确定线程池的大小需要考虑CPU数量/内存大小等因素，在*《Java Concurrency in Practice》*一书中给出了一个估算线程池大小的经验公式：
$$
N_{CPU}=CPU数量
$$

$$
U_{CPU} = 目标CPU的使用率，0<U_{CPU}<1
$$

$$
W/C = 等待时间与计算时间的比率
$$

为保持处理器达到期望的使用率，最优的池的大小等于：
$$
N_{threads} = N_{CPU}*U_{CPU}*(1+W/C)
$$

> 获取CPU数量：`Runtime.getRuntime().availableProcessors()`



线程池状态







如果运行的线程小于corePoolSize，则尝试使用给定的命令作为第一个任务来启动新线程。对addWorker的调用会自动地检查runState和workerCount，从而通过返回false来防止在不应该添加线程的情况下添加错误警报。



如果一个任务可以成功地排队，那么我们仍然需要再次检查是否应该添加一个线程(因为现有的线程在最后一次检查后死亡)，或者池在进入这个方法后关闭。因此，我们重新检查状态，如果有必要，如果停止，则回滚排队;如果没有，则启动新线程。



如果无法对任务排队，则尝试添加新线程。如果它失败了，我们知道我们被关闭或饱和，因此拒绝任务。

    /*
        * Proceed in 3 steps:
        *
        * 1. If fewer than corePoolSize threads are running, try to
        * start a new thread with the given command as its first
        * task.  The call to addWorker atomically checks runState and
        * workerCount, and so prevents false alarms that would add
        * threads when it shouldn't, by returning false.
        *
        * 2. If a task can be successfully queued, then we still need
        * to double-check whether we should have added a thread
        * (because existing ones died since last checking) or that
        * the pool shut down since entry into this method. So we
        * recheck state and if necessary roll back the enqueuing if
        * stopped, or start a new thread if there are none.
        *
        * 3. If we cannot queue task, then we try to add a new
        * thread.  If it fails, we know we are shut down or saturated
        * and so reject the task.
        */
在将来的某个时候执行给定的任务。任务可以在新线程中执行，也可以在现有的池线程中执行。

如果无法提交任务执行，可能是因为此执行程序已关闭，也可能是因为其容量已达到，则由当前{@link RejectedExecutionHandler}处理该任务。

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```



