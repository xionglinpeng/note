# Exchanger

## Exchanger

Exchanger用于线程间的数据交换，它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过`exchange`方法交换数据，如果第一个线程先执行`exchange()`方法，他会一直等待第二个线程也执行`exchange`方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

*示例：*

```java
private static final Exchanger<String> exchanger = new Exchanger<>();

public static void main(String[] args) {
    new Thread(()->{
        String A = "a";
        try {
            exchanger.exchange(A);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    new Thread(()->{
        String B = "b";
        try {
            String A = exchanger.exchange(B);
            System.out.printf("A和B数据是否一致:%s,A=%s;B=%s\n",A.equals(B),A,B);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
}
```

`exchange()`方法可以响应中断。如果一个线程（A）执行了`exchange()`方法交换数据，但是一直没有等到另一个线程（B）执行`exchange()`方法交换数据，那么A线程将一直阻塞，可以使用`exchange(V x, long timeout, TimeUnit unit)`设置超市时间，避免无限期阻塞等待。

Exchanger可以用于两个线程之间交换数据，如果同时存在两个以上（偶数个）的线程交换数据将产生混乱（并不是绝对的，在某些特殊的场合下是符合要求的）。

示例：

```java
private static final Exchanger<String> exchanger = new Exchanger<>();

public static void main(String[] args) {
    new Thread(()->{
        String A = "a";
        try {
            exchanger.exchange(A);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    new Thread(()->{
        String B = "b";
        try {
            String A = exchanger.exchange(B);
            System.out.printf("A和B数据是否一致:%s,A=%s;B=%s\n",A.equals(B),A,B);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    new Thread(()->{
        String C = "C";
        try {
            exchanger.exchange(C);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    new Thread(()->{
        String D = "D";
        try {
            exchanger.exchange(D);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
}
```

多执行几次，其输出的结果不是固定的，一会儿是`A=a`，一会儿是`A-c`。

## API

`public V exchange(V x) throws InterruptedException`

`public V exchange(V x, long timeout, TimeUnit unit) throws InterruptedException, TimeoutException`