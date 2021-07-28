# RateLimiter

Guava框架提供了令牌桶算法的实现，可直接拿来使用，使用Guava框架的RateLimiter类即可创建一个令牌桶限流器，比如设置每秒放置令牌数为5个，那么RateLimiter对象可以保证1秒内不会放入超过5个令牌，并且以固定的速率进行放置令牌，达到平滑输出的效果。

**Ratelimiter实现令牌桶限流的特点**

1. RateLimiter使用令牌桶算法，会进行令牌的累积，如果获取令牌的频率比较低，则不会导致等待，直接获取令牌。

2. RateLimiter由于会累积令牌，所以可以应对突发流量，比如同时请求5个令牌，由于此时令牌桶中有累积的令牌，足以快速响应。

3. RateLimiter在没有足够令牌发放时，采用滞后处理的方式，也就是前一个请求获取令牌所需等待的时间由下一次请求来承受，也就是代替前一个请求进行等待。

**Ratelimiter的可用函数**

RateLimiter通过`create()`方法创建一个令牌桶限流器，提供了两个公共的create方法：

- `public static RateLimiter create(double permitsPerSecond)`：
- `public static RateLimiter create(double permitsPerSecond, long warmupPeriod, TimeUnit unit)`：

参数：

- `permitsPerSecond`：表示两个含义，一是令牌桶的大小，即最大容量；二则表示令牌桶每秒可以生成多个令牌。
- `warmupPeriod`：预热时间，表示RateLimiter在达到稳定(最大)速率之前提高其速率的持续时间（这个参数的意义不太好理解，请参看下面的“预热”一节）。
- `unit`：预热时间单位。

通过`acquire()`方法获取令牌：

- `public double acquire()`：每次从桶中获取一个令牌，如果桶中没有令牌，则阻塞等待。等价于`acquire(1)`。
- `public double acquire(int permits)`：每次从桶中获取指定数量的令牌，如果桶中没有令牌，则阻塞等待。

还可以通过`tryAcquire()`方法获取令牌：

- `public boolean tryAcquire()`：尝试从令牌桶中获取令牌，并立即返回，如果可以获取，返回true，否则返回false。等价于`tryAcquire(1)`。
- `public boolean tryAcquire(int permits)`：尝试从令牌桶中获取指定数量的令牌，并立即返回，如果可以获取，返回true，否则返回false。
- `public boolean tryAcquire(long timeout, TimeUnit unit)`：在不超时的情况下，尝试从令牌桶中获取令牌，如果超时仍没有获取到，则立即返回false（不等待）。等价于`tryAcquire(1, timeout, unit)`。
- `public boolean tryAcquire(int permits, long timeout, TimeUnit unit)`：在不超时的情况下，尝试从令牌桶中获取指定数量的令牌，如果超时仍没有获取到，则立即返回false（不等待）。

设置和获取令牌桶的速率：

- `public final void setRate(double permitsPerSecond)`：设置令牌桶放置令牌的速率，permitsPerSecond/s。
- `public final double getRate()`：获取令牌桶的速率。

---

**Ratelimiter的限流器创建**

创建令牌桶限流器时，对于通过`permitsPerSecond`声明令牌桶大小，还有另一个含义需要注意，`permitsPerSecond`不仅仅表示令牌桶的大小，它还表示了令牌桶每秒可以生成的令牌数，例如将`permitsPerSecond`设置为5，表示在1秒内可以生成5个令牌，即平均每200毫秒生成一个令牌，而不是在1秒的最后一刻一次性生成5个令牌。

RateLimiter的这种实现将突发的请求转换为了平滑的请求。

如下代码所示：

```java
public static void main(String[] args) {
    //表示桶容量为5，且每秒新增5个令牌
    RateLimiter limiter = RateLimiter.create(5);
    for (int i = 0; i < 10; i++) {
        //返回值表示从令牌桶中获取一个令牌所花费的时间，但是为秒
        System.out.println(limiter.acquire());
    }
}
```

输出结果：

```
0.0
0.197499
0.194651
0.197918
0.197782
0.199493
0.199004
0.198542
0.199335
0.198839
```

从结果可以看到，当第一次获取令牌的时候，因为桶中已经存在了令牌，因此可以直接获取，并且时间为0.0。当后续获取的时候，因为桶中没有令牌，所以等待了接近200毫秒才获取到。

---

**预热**

预热的作用是用于在RateLimiter刚创建时，桶中还没有任何令牌，这时需要在指定的预热期限内添加令牌，其添加的速率开始很慢，但它越来越快，直到达到每秒的最大速率。

例如下面的代码：

```java
RateLimiter limiter = RateLimiter.create(10, Duration.ofSeconds(2));
```

设定的预热时间为2秒，表示要在2秒内完成预热，即2秒内的完成最初10个令牌的增加，但最初始的10个令牌增加的速率不是平均的，而是一个预热过程，即速率最开始很大，然后越来越快。譬如第二个令牌的准备时间为300毫秒（第一个令牌创建时就准备好了），第二个为200毫秒，第三个为100毫秒，...，以此类推。当预热完成之后，达到最大速率，例如上面的例子，虽然预热时间为2秒，但最大速率却是需要1秒内准备10个令牌，换算一下即平均每个令牌的准备时间需要100毫秒，即最大速率为100毫秒。请看下面的代码：

```java
@SuppressWarnings("UnstableApiUsage")
public static void main(String[] args) {
    //表示桶容量为10，预热时间为2s，即最开始10个需要2秒准备，后续需要每秒准备10个
    RateLimiter limiter = RateLimiter.create(10, Duration.ofSeconds(2));
    for (int i = 0; i < 100; i++) {
        //返回值表示从令牌桶中获取一个令牌所花费的时间，但是为秒
        System.out.println(limiter.acquire());
    }
}
```

下面是输出的结果：

```
0.0
0.288894
0.264122
0.247313
0.228972
0.209553
0.188927
0.169644
0.148875
0.129387
0.108598
0.099064
......
0.09718
```

通过以上结果可以看出，第一个令牌直接在RateLimiter创建时就已经准备好了，因此它的准备时间为0。第二个令牌的准备时间为0.28秒，第三个为0.26秒，第四个为0.24秒，...，一直到第10个0.12秒；第11个是就是0.10了，刚好达到1秒10个的平均速率，即最大速率100毫秒。

---

**RateLimiter应对突发请求**

RateLimiter可以应对突发请求，即突然一瞬间来了超出桶容量限制的请求数时，RateLimiter也能出处理，即可以提前借出令牌，但是借出的令牌是需要还回来的，因此需要后续请求弥补。

如下代码所示：

```java
public static void main(String[] args) {
    //表示桶容量为5，且每秒新增5个令牌
    RateLimiter limiter = RateLimiter.create(5);
    //1秒5个，50个=10秒
    System.out.println(limiter.acquire(50));
    for (int i = 0; i < 4; i++) {
        //返回值表示从令牌桶中获取一个令牌所花费的时间，但是为秒
        System.out.println(limiter.acquire(5));
    }
}
```

输出结果：

```
0.0
9.997985
0.995756
0.997871
0.998841
```

如上代码所示，RateLimiter创建了一个大小为5的令牌桶，第一次获取令牌的时候，一次性获取了50个令牌，从结果可以看出，它是获取成功了。但是桶最大为5个，因此超出的令牌需要后续弥补，从结果可以看到，但第二次获取令牌的时候，等待了10秒，弥补了之前借出的50个令牌。



> Note : 使用注解`@SuppressWarnings("UnstableApiUsage")`可以去除RateLimiter的警告。







