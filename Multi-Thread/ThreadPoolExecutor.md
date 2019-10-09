```
ThreadPoolExecutor
```









```java
ThreadPoolExecutor(int corePoolSize,
                   int maximumPoolSize,
                   long keepAliveTime,
                   TimeUnit unit,
                   BlockingQueue<Runnable> workQueue)
ThreadPoolExecutor(int corePoolSize,
                   int maximumPoolSize,
                   long keepAliveTime,
                   TimeUnit unit,
                   BlockingQueue<Runnable> workQueue,
                   ThreadFactory threadFactory)
ThreadPoolExecutor(int corePoolSize,
                   int maximumPoolSize,
                   long keepAliveTime,
                   TimeUnit unit,
                   BlockingQueue<Runnable> workQueue,
                   RejectedExecutionHandler handler)
ThreadPoolExecutor(int corePoolSize,
                   int maximumPoolSize,
                   long keepAliveTime,
                   TimeUnit unit,
                   BlockingQueue<Runnable> workQueue,
                   ThreadFactory threadFactory,
                   RejectedExecutionHandler handler)
```



- `int corePoolSize`：线程池的核心线程数。
- `int maximumPoolSize`：线程池的最大线程数。
- `long keepAliveTime`：线程池中空闲线程的存活时间。
- `TimeUnit unit`：空闲线程存活时间的单位。
- `BlockingQueue<Runnable> workQueue`：用于存放任务的阻塞队列。
- `ThreadFactory threadFactory`：线程池创建线程的工厂。
- `RejectedExecutionHandler handler`：饱和策略。



线程池实现原理



当调用线程池的`execute()`方法添加任务时，如果线程数小于核心线程数，则直接创建一个新的工作线程来执行任务；如果核心线程数满了，就将任务添加到阻塞队列；如果阻塞队列也满了，就创建新的线程执行任务，直到最大线程数据为止；如果最大线程数也满了，将执行饱和策略。



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

合理配置线程池

线程池的大小并不是越大越好，需要根据具体的情况合理的配置大小，可以从以下几个角度去考虑：

- 任务的性质：CPU密集型任务/IO密集型任务/混合型任务
- 任务的优先级：高/中/低
- 人物的执行时间：长/中/短
- 任务的依赖性：是否依赖其他系统资源，如数据库连接。

CPU密集型任务应配置尽可能小的线程数，一般配置N~cpu~+1个线程数。==??==

IO密集型任务应配置尽可能多的线程数，一般配置2*N~cpu~个线程数。==??==

混合型任务，如果可以拆分，可以将其拆分成一个CPU密集型任务和一个IO密集型任务，如果两个任务的执行时间相差不大，那么分解后执行的吞吐量将高于串行执行的吞吐量。如果两个任务的执行时间相差太大，则没必要进行分解。

>获取CPU数量：`Runtime.getRuntime().availableProcessors()`

优先级不同的任务可以使用优先级队列`PriorityBlockingQueue`来处理。它可以让优先级高的任务先执行。

执行时间不同的任务可以交给不同规模的线程池来处理，或者可以使用优先级队列，让执行时间短的任务先执行。

依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，等待的时候越长，则CPU空闲时间就越长，那么线程数应该设置的越大，这样才能更好的利用CPU。



