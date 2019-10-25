# CyclicBarrier

CyclicBarrier意为循环栅栏，它可以控制阶段性的任务，比如有10个人需要执行一个相同的任务，我们可以为这10个人分别开启一个线程，让他们同时执行，但是有一个要求：这个任务是分阶段的，分为3个阶段，要求这个10个人都完成第一个阶段之后，才可以开始第二阶段的任务，同时，都完成第二阶段任务之后，才可以开始第三阶段的任务。

*示例1：*

司令下达命令，要求10个士兵一起去完成一项任务，要求他们先集合完毕，然后才一起雄赳赳气昂昂去执行任务。

```java
public static void main(String[] args) {
    CyclicBarrier cyclic = new CyclicBarrier(10);

    for (int i = 1; i <= 10; i++) {
        int k = i;
        Thread thread = new Thread(()->{
            try {
                Thread.sleep(new Random().nextInt(1000));
                System.out.printf("士兵%d集合报道\n",k);
                cyclic.await();
                //休眠，模拟执行任务
                Thread.sleep(new Random().nextInt(1000));
                System.out.printf("报告：士兵%d任务执行完成完毕\n",k);
                cyclic.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        thread.start();
    }
}
```

输出结果：

```
士兵3集合报道
士兵4集合报道
士兵8集合报道
士兵5集合报道
士兵1集合报道
士兵7集合报道
士兵2集合报道
士兵6集合报道
士兵9集合报道
士兵10集合报道
报告：士兵7任务执行完成完毕
报告：士兵6任务执行完成完毕
报告：士兵3任务执行完成完毕
报告：士兵5任务执行完成完毕
报告：士兵2任务执行完成完毕
报告：士兵10任务执行完成完毕
报告：士兵1任务执行完成完毕
报告：士兵4任务执行完成完毕
报告：士兵8任务执行完成完毕
报告：士兵9任务执行完成完毕
```

可以看到，士兵们都集合完成之后才开始执行任务，最后，任务都执行完时，才报告任务执行完毕。



CyclicBarrier有两个构造函数

```java
public CyclicBarrier(int parties) {
	this(parties, null);
}

public CyclicBarrier(int parties, Runnable barrierAction) {
	...
}
```

有两个参数：

1. parties：计数总数。
2. barrierAction：当计数器完成一次计数后的回调（每次完成都会回调）。

如果barrierAction，当每一个阶段性的任务完成之后，都会进行回调。仍以*示例1*进行说明，如果我们希望当士兵都集合完毕以及任务执行完毕之后，司令向上级报告任务进度，就可以是`barrierAction`来完成。

*示例2：*

```java
public static int tag = 1;

public static void main(String[] args) {
    CyclicBarrier cyclic = new CyclicBarrier(10,()->{
        if (tag == 1) {
            System.out.println("报告首长：士兵集合完毕，请指示。");
            tag = 2;
        } else if (tag == 2){
            System.out.println("报告首长：任务完成，请指示。");
            tag = 3;
        }
    });

    for (int i = 1; i <= 10; i++) {
        int k = i;
        Thread thread = new Thread(()->{
            try {
                Thread.sleep(new Random().nextInt(1000));
                System.out.printf("士兵%d集合报道\n",k);
                cyclic.await();
                //休眠，模拟执行任务
                Thread.sleep(new Random().nextInt(1000));
                System.out.printf("报告：士兵%d任务执行完成完毕\n",k);
                cyclic.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        thread.start();
    }
}
```

输出结果：

```
士兵3集合报道
士兵9集合报道
士兵5集合报道
士兵10集合报道
士兵7集合报道
士兵2集合报道
士兵8集合报道
士兵1集合报道
士兵6集合报道
士兵4集合报道
报告首长：士兵集合完毕，请指示。
报告：士兵7任务执行完成完毕
报告：士兵9任务执行完成完毕
报告：士兵10任务执行完成完毕
报告：士兵5任务执行完成完毕
报告：士兵6任务执行完成完毕
报告：士兵4任务执行完成完毕
报告：士兵2任务执行完成完毕
报告：士兵8任务执行完成完毕
报告：士兵3任务执行完成完毕
报告：士兵1任务执行完成完毕
报告首长：任务完成，请指示。
```

可以看到，结果符合我们预期。

## CyclicBarrier API

1. `public int await()`：等待任务节点完成。
2. `public int await(long timeout, TimeUnit unit)`：限时等待任务节点完成。
3. `public void reset()`：将循环栅栏重置为初始状态，如果有任何线程正在屏障处等待，它们将响应抛出屏障破损的异常（`BrokenBarrierException`）。如果是在因为其他原因（例如中断）造成CyclicBarrier 破损之后，调用reset()复位重置，那么这种情况将比较复杂，线程需要以其他方式重新同步，并选择一种方式执行重置。最好是为后续使用创建一个新的屏障。
4. `public int getParties()`：获取计数总数
5. `public int getNumberWaiting()`：获取等待任务总数
6. `public boolean isBroken()`：CyclicBarrier 是否破损

## 异常

CyclicBarrier 的`await()`可以会抛出两个异常，分别是`InterruptedException`、`BrokenBarrierException`。

- InterruptedException：表示等待过程中可以响应中断

- BrokenBarrierException：出现这个异常，表示CyclicBarrier已经破损了，这里的破损指的是CyclicBarrier在执行任务过程当中，因为异常原因，导致一个或多个`await()`方法不能正常等待，从而不能完成计数，而导致其他正常的`await()`方法永久阻塞，为避免这种情况而抛出的异常。

*示例3：*

下面使用一个示例来模拟其中一个任务中断的情况。

```java
public static void main(String[] args) {
    CyclicBarrier cyclic = new CyclicBarrier(10);

    List<Thread> threads = new ArrayList<>(10);
    for (int i = 1; i <= 10; i++) {
        int k = i;
        threads.add(new Thread(()->{
            try {
                Thread.sleep(new Random().nextInt(1000));
                System.out.printf("士兵%d集合报道\n",k);
                cyclic.await();
                //休眠，模拟执行任务
                Thread.sleep(new Random().nextInt(1000));
                System.out.printf("报告：士兵%d任务执行完成完毕\n",k);
                cyclic.await();
            } catch (InterruptedException e) {
                System.out.printf("CyclicBarrier%s破损..\n",cyclic.isBroken()?"已经":"没有");
                System.out.printf("等待士兵数%d\n",cyclic.getNumberWaiting());
                System.out.printf("士兵%d出现意外，不能报道\n",k);
            } catch (BrokenBarrierException e) {
                System.out.printf("士兵%d报告，报道不能进行，请求打道回府\n",k);
            }
        }));
    }
    threads.forEach(Thread::start);
    //随机使任意线程中断
    int m = new Random().nextInt(10);
    for (int i = 1; i <= threads.size(); i++) {
        if (i == m) {
            try {
                Thread.sleep(new Random().nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            threads.get(i-1).interrupt();
            break;
        }
    }
}
```

输出结果如下：

```
士兵5集合报道
士兵2集合报道
士兵4集合报道
士兵8集合报道
士兵1集合报道
士兵3集合报道
士兵9集合报道
士兵4报告，报道不能进行，请求打道回府
CyclicBarrier已经破损..
等待士兵数0
士兵8出现意外，不能报道
士兵9报告，报道不能进行，请求打道回府
士兵3报告，报道不能进行，请求打道回府
士兵1报告，报道不能进行，请求打道回府
士兵2报告，报道不能进行，请求打道回府
士兵5报告，报道不能进行，请求打道回府
士兵6集合报道
士兵6报告，报道不能进行，请求打道回府
士兵7集合报道
士兵7报告，报道不能进行，请求打道回府
士兵10集合报道
士兵10报告，报道不能进行，请求打道回府
```

从结果可以分析出，士兵8所在的线程被中断了，打印出“士兵8出现意外，不能报道”，除了士兵8所在的线程响应了中断异常意外，其他的线程都响应了`BrokenBarrierException`异常，即输出了“打道回府”的信息。

上面的代码是有BUG的，当我们多执行几次，可以会输入如下结果，并且不能正常结束。

BUG情况下输出结果：

```
士兵8集合报道
士兵10集合报道
士兵5集合报道
士兵2集合报道
士兵9集合报道
士兵6集合报道
士兵1集合报道
士兵4集合报道
士兵7集合报道
CyclicBarrier没有破损..
等待士兵数9
士兵3出现意外，不能报道
```

可以看到，打印的信息很奇怪："CyclicBarrier没有破损.."，正常情况下CyclicBarrier是肯定破损的，为何会如此呢？需要注意到，上述代码不仅仅CyclicBarrier的`await()`方法会响应中断异常，`Thread.sleep()`也会响应中断异常，如果是Thread的`sleep()`方法响应了中断，那么CyclicBarrier是不会知道其中一个线程被中断了，仍然认为一切正常，其他正常的线程仍然傻傻的继续等待，但是却永远不会有等到。

## reset

验证一下`reset()`重置初始化的情况。

*示例4：*

```java
public static void main(String[] args) throws InterruptedException {
    CyclicBarrier cyclic = new CyclicBarrier(10);

    for (int i = 1; i <= 10; i++) {
        int k = i;
        Thread thread = new Thread(()->{
            try {
                Thread.sleep(new Random().nextInt(1000));
                System.out.printf("士兵%d集合报道\n",k);
                cyclic.await();
                //休眠，模拟执行任务
                Thread.sleep(new Random().nextInt(1000));
                System.out.printf("报告：士兵%d任务执行完成完毕\n",k);
                cyclic.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                System.out.printf("士兵%d报告，报道不能进行，请求打道回府\n",k);
            }
        });
        thread.start();
    }

    Thread.sleep(new Random().nextInt(1000));
    System.out.println(cyclic.getNumberWaiting());
    cyclic.reset();
    System.out.println(cyclic.getNumberWaiting());
    Thread.sleep(new Random().nextInt(2000));
    System.out.println(cyclic.getNumberWaiting());
}
```

输出结果如下（这只是情况之一）：

```
士兵9集合报道
士兵5集合报道
士兵6集合报道
士兵2集合报道
士兵4集合报道
5
0
士兵9报告，报道不能进行，请求打道回府
士兵6报告，报道不能进行，请求打道回府
士兵2报告，报道不能进行，请求打道回府
士兵5报告，报道不能进行，请求打道回府
士兵4报告，报道不能进行，请求打道回府
士兵10集合报道
士兵1集合报道
士兵7集合报道
士兵8集合报道
士兵3集合报道
5
```

从结果可以分析得出，首先在复位之前，有5个线程在等待，复位之后，等待线程变为0，并且之前等待的线程响应`BrokenBarrierException`异常。紧接着，后续的5个线程继续执行（总共有10个线程）并等待，最后输出等待的线程数量是5，这5个等待的线程是在之前复位之后才执行的，所以上述示例测试阻塞，不能正常结束。

