# Guava Cache

## 范例

```java
//缓存接口这里是LoadingCache，LoadingCache在缓存项不存在时可以自动加载缓存
//CacheBuilder的构造函数是私有的，只是通过其静态方法newBuilder()来获得CacheBuilder的实例
LoadingCache<Integer, String> cache = CacheBuilder.newBuilder()
    //设置并发级别为8，并发级别是指可以同时写缓存的线程数
    .concurrencyLevel(8)
    //设置缓存容器的初始容量为10
    .initialCapacity(10)
    //设置缓存最大容量为100，超过100之后就会按照LRU最近最少使用算法来移除缓存
    .maximumSize(100)
    //设置写缓存后10秒过期：缓存项在给定时间内没有被写访问（创建或覆盖）
    .expireAfterWrite(10, TimeUnit.SECONDS)
    //设置写缓存后10秒过期：缓存项在给定时间内没有被读/写访问
    .expireAfterAccess(10, TimeUnit.SECONDS)
    //使用弱引用的存储键，当键没有其他（强或软）引用时，缓存项可以被垃圾回收。
    .weakKeys()
    //使用弱引用的存储值，当键没有其他（强或软）引用时，缓存项可以被垃圾回收。
    .weakValues()
    //使用软引用的存储值
    .softValues()
    //设置缓存的移除通知
    .removalListener(notification -> System.out.println("移除原因：" + notification.getCause()))
    //设置要统计的缓存命中率
    .recordStats()
    //设置自动刷新：每10分钟刷新一次
    .refreshAfterWrite(10, TimeUnit.MINUTES)
    //设置自定义时间源（基于时间对缓存操作的时间源，例如缓存过期）
    .ticker(new Ticker() {
        @Override
        public long read() {
            return 0;
        }
    })
    //设置最大权重100
    .maximumWeight(100)
    //设置权重函数
    .weigher((Weigher<Integer, String>) (key, value) -> {
        return RandomUtils.nextInt(1,100); //伪代码->给了一个随机数
    })
    //build方法可以指定CacheLoader，在缓存不存在时通过CacheLoader的实现自动加载缓存
    .build(new CacheLoader<>() {
        @Override
        public String load(Integer key) throws Exception {
            return RandomStringUtils.random(10); //伪代码->给了一个10位长度的随机字符串
        }
    });
```

## 适用性

缓存在很多场景下都是相当有用的。例如，计算或检索一个值的代价很高，并且对同样的输入需要不止一次获取值的时候，就应当考虑使用缓存。

Guava Cache与`ConcurrentMap`很相似，但也不完全一样。最基本的区别是`ConcurrentMap`会一直保持所有添加的元素，直到显式的移除。相对地，Guava Cache为了限制内存占用，通常都设定为自动回收元素。在某些场景下，尽管`LoadingCache`不回收元素，它也是很有用的，因为它会自动加载缓存。

通常来说，Guava Cache适用于：

- 你愿意消耗一些内存空间来提升速度。
- 你预料到某些键会被查询一次以上。
- 缓存中存在的数据总量不会超出内存容量。

> Note：如果你不需要Cache中的特性，使用`ConcurrentMap`有更好的内存效率——但Cache的大多数特性都很难基于旧有的`ConcurrentMap`复制，甚至根本不可能做到。

## 加载

在使用缓存前，首先问自己一个问题，有没有合理的默认方法来加载或计算与键关联的值？如果有的话，你应当使用`CacheLoader`。如果没有，或者你想要覆盖默认的加载运算，同时保留“**获取缓存 - 如果没有 - 则计算**”[get-if-absent-compute]的原子语义，你应该在调用`get`时传入一个`Callable`实例，缓存元素也可以通过`Cache.put`方法直接插入，但自动加载是首选的，因为它可以更容易地推断所有缓存内容的一致性。

### CacheLoader

`LoadingCache`是附带`CacheLoader`构建而成的缓存实现。创建自己的`CacheLoader`通常只需要简单地实现`V load(K key) throws Exception`方法。例如，你可以用下面的代码构建`LoadingCache`：

```java
LoadingCache<Integer, String> cache = CacheBuilder.newBuilder()
    .maximumSize(100)
    .build(new CacheLoader<>() {
        @Override
        public String load(Integer key) throws Exception {
            return RandomStringUtils.random(10);
        }
    });
```

另外`CacheLoader`还提供了from静态函数以创建`CacheLoader`，分别是：

- `public static <V> CacheLoader<Object, V> from(Supplier<V> supplier)`
- `public static <K, V> CacheLoader<K, V> from(Function<K, V> function)`

分别提供了Supplier和Function，对于Supplier而言，只需要给以返回值，没有参数；而Function有参数（key）以及返回值。

```java
//Function
CacheBuilder.newBuilder()
    .build(CacheLoader.from(key->{
        return null;
    }));
//Supplier
CacheBuilder.newBuilder()
    .build(CacheLoader.from(()->{
        return null;
    }));
```

从`LoadingCache`查询的正规方式是使用`get(K)`方法。这个方法要么返回已经缓存的值，要么使用`CacheLoader`通向缓存原子地加载新值。由于`CacheLoader`可能抛出异常，`LoadingCache.get(K)`也声明为抛出`ExecutionException`异常。如果你定义的`CacheLoader`没有声明任何检查型异常，则可以通过`getUnchecked(K)`查询缓存；但必须注意，一旦`CacheLoader`声明了检查型异常，就不可以调用`getUnchecked(K)`。

```java
cache.getUnchecked(K)
```

`ImmutableMap<K, V> getAll(Iterable<? extends K> keys) throws ExecutionException;`方法用来执行批量查询。默认情况下，对每个不存在缓存中的键，`getAll`方法会单独调用`CacheLoader.load`来加载缓存项。如果批量的加载比多个单独加载更高效，你可以重载`CacheLoader.loadAll`来利用这一点。`getAll(Iterable)`的性能也会相应提升。

> 注：`CacheLoader.loadAll`的实现可以为没有明确请求的键加载缓存值。例如，为某组中的任意键计算值时，能够获取该组中的所有键值。`loadAll`方法就可以实现在为同一时间获取该组的其他键值。校注：`getAll(Iterable<? extends K>)`方法会调用`loadAll`，但会筛选结果，只会返回请求的键值对。

### Callable

所有类型的Guava Cache，不管有没有自动加载功能，都支持`get(K, Callable<V>)`方法。这个方法返回缓存中相应的值，或者用给定的Callable运行并把结果放入到缓存中。在整个加载方法完成前，缓存项相关的可观察状态都不会改变。这个方法简便地实现了模式“**如果有缓存则返回；否则运算、缓存、然后返回**”。

```java
String value = cache.get(int, new Callable<String>() {
    @Override
    public String call() throws Exception {
        return null;
    }
});
```

### 显示插入

使用`Cache.put(K, V)`方法可以直接向缓存中插入值，这会直接覆盖掉给定键之前映射的值。使用`Cache.asMap()`视图提供的任何方法也能修改缓存。但请注意，`asMap`视图的任何方法都不能保证缓存项被原子地加载到缓存中。进一步说，`asMap`视图的原子运算在Guava Cache的原子加载范围之外，所以相比于`Cache.asMap().putIfAbsent()`，`Cache.get(K, Callable<V>)`应该总是优先使用。

## 缓存回收

一个残酷的现实是，我们几乎一定没有足够的内存来缓存所有数据。你必须决定：什么时候某个缓存项就不值得保留了？Guava Cache提供了三种基本的缓存回收方式：**基于容量回收**、**定时回收**、**基于引用回收**。

### 基于容量的回收

如果要规定缓存项的数目不能超过固定值，只需要使用`CacheBuilder.maximumSize(long)`。缓存将尝试回收最近没有使用或总体上很少使用的缓存项。

> Note：在缓存项的数目达到限定值之前，缓存就可能进行回收操作——通常来说，这种情况发生在缓存项的数据逼近限定值时。

另外，不同的缓存项有不同的“权重”（weights）。例如，如果你的缓存值，占据完全不同的内存空间，你可以使用`CacheBuilder.weigher(Weigher)`指定一个权重函数，并且用`CacheBuilder.maximumWeight(long)`指定最大权重。在权重限定的场景中，除了要注意回收也是在权重逼近限定值时就进行了，还要知道最大权重是在缓存创建时计算的，因此要考虑最大权重计算的复杂度。

```java
LoadingCache<Integer, String> cache = CacheBuilder.newBuilder()
    //设置缓存最大容量为100，超过100之后就会按照LRU最近最少使用算法来移除缓存
    .maximumSize(100)
    //设置最大权重100
    .maximumWeight(100)
    //设置权重函数
    .weigher((Weigher<Integer, String>) (key, value) -> {
        return RandomUtils.nextInt(1,100); //伪代码->给了一个随机数
    })
    //build方法可以指定CacheLoader，在缓存不存在时通过CacheLoader的实现自动加载缓存
    .build(new CacheLoader<>() {
        @Override
        public String load(Integer key) throws Exception {
            return RandomStringUtils.random(10); //伪代码->给了一个10位长度的随机字符串
        }
    });
```

### 定时回收

`CacheBuilder`提供了两种定时回收的方法：

- `expireAfterAccess(long duration, TimeUnit unit)`：缓存项在给定时间内没有被**读/写访问**，则回收。请注意这种缓存的回收顺序和基于大小回收一样。
- `expireAfterWrite(long duration, TimeUnit unit)`：缓存项在给定时间内没有被**写访问**（创建或覆盖），则回收。如果认为缓存数据总是在固定时间后变得陈旧不可用，那么这种回收方式是可取的。



### 基于引用的回收

通过使用弱引用的键、或弱引用的值、或软引用的值，Guava Cache可以把缓存设置为允许垃圾回收：

- `CacheBuilder.weakKeys()`：使用弱引用的存储键。当键没有其他（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式（==），使用弱引用键的缓存用==而不是equals比较键。
- `CacheBuilder.weakValues()`：使用弱引用的存储值。当值没有其他（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式（==），使用弱引用值的缓存用==而不是equals比较键。
- `CacheBuilder.softValues()`：使用软引用存储值。软引用只有在响应内存需要时，才按照全局最近最少使用的顺序回收。考虑到使用软引用的性能影响，我们通常建议使用更有性能预测性的缓存大小限定（基于容量回收）。使用软引用值的缓存用==而不是equals比较键。

### 显式清除

任何时候，你都可以显式地清除缓存项，而不是等到它被回收：

- 个别清除：`invalidate(Object key)`
- 批量清除：`invalidateAll(Iterable<?> keys)`
- 清除全部：`invalidateAll()`

### 移除监听器

通过` CacheBuilder.removalListener(RemovalListener)`，你可以声明一个监听器，以便缓存项被移除时做一些额外的操作。缓存项被移除时，`RemovalListener`会获取移除通知[`RemovalNotification`]，其中包含移除原因[`RemovalCause`]、键和值。

请注意，`RemovalListener`抛出的任何异常都会记录到日志后被丢弃。

```java
LoadingCache<Integer, String> cache = CacheBuilder.newBuilder()
    .removalListener(notification -> System.out.println("移除原因：" + notification.getCause()))
    .build(...);
```

> Note：默认情况下，监听器方法是在移除缓存时同步调用的。因为缓存的维护和请求响应通常是同时进行的，代价高昂的监听器方法在同步模式下会拖慢正常的缓存请求。在这种情况下，你可以使用`RemovalListeners.asynchronous(RemovalListener, Executor)`把监听器装饰为异步操作。

### 清理什么时候发生

使用`CacheBuilder`构建的缓存不会“自动”执行清理和回收工作，也不会在某个缓存项过期后马上清理，也没有诸如此类的清理机制。相反，它会在写操作时顺带做少量的维护工作，或者偶尔在读操作时做——如果写操作是在太少的话。

这样做的原因在于：如果要自动地持续清理缓存，就必须有一个线程，这个线程会和用户操作竞争共享锁。此外，某些环境下线程创建可能受限制，这样`CacheBuilder`就不可用了。

相反，我们把选择权交到你手里。如果你的缓存是高吞吐的，那就无需担心缓存的维护和清理等工作。如果你的缓存只会偶尔有写操作，而你又不想清理工作阻碍读操作，那么可以创建自己的维护线程，以固定的时间间隔调用`Cache.cleanUp()`。`ScheduledExecutorService`可以帮助你很好地实现这样的定时调度。

### 刷新

刷新和回收不太一样。正如`LoadingCache.refresh(K)`所声明，刷新表示为键加载新值，这个过程可以是异步的。在刷新操作进行时，缓存仍然可以向其他线程返回旧值。而不像回收操作，读缓存的线程必须等待新值加载完成。

如果刷新过程抛出异常，缓存将保留旧值，而异常会在记录到日志后被丢弃。

重载`CacheLoader.reload(K key, V oldValue)`可以扩展刷新时的行为，这个方法允许开发者在计算新值时使用旧的值。

```java
LoadingCache<Integer, String> cache = CacheBuilder.newBuilder()
    .maximumSize(100)
    //设置自动刷新：每10分钟刷新一次
    .refreshAfterWrite(10, TimeUnit.MINUTES)
    .build(new CacheLoader<>() {
        @Override
        public String load(Integer key) throws Exception {
            return RandomStringUtils.random(10);
        }
        @Override
        public ListenableFuture<String> reload(Integer key, String oldValue) throws Exception {
            return super.reload(key, oldValue);
        }
    });
```

`CacheBuilder.refreshAfterWrite(long duration, TimeUnit unit)`可以为缓存增加自动定时刷新功能。和`expireAfterWrite`相反，`refreshAfterWrite`通过定时刷新可以让缓存项保持可用。但请注意：缓存项只有在被检索时才会真正刷新（如果`CacheLoader.refresh`实现为异步，那么检索不会被刷新拖慢）。因此，如果你在缓存上同时声明`expireAfterWrite`和`refreshAfterWrite`，缓存并不会因为刷新盲目地定时重置，如果缓存项没有被检索，那刷新就不会真的发生，缓存项在过期时间后也变得可以回收。

## 其他特性

### 统计

`CacheBuilder.recordStats()`用来开启Guava Cache的统计功能。统计打开后，`cache.stats()`方法会返回`CacheStats`方法提供如下统计信息：

- `hitRate()`：缓存命中率；
- `averageLoadPenalty()`：加载新值的平均时间，单位为纳秒；
- `evictionCount()`：缓存项被回收的总数，不包括显式清除；

此外，还有其他很多统计信息。这些统计信息对于调整缓存设置是至关重要的，在性能要求高的应用中我们建议密切关注这些数据。

### asMap视图

asMap视图提供了缓存的`ConcurrentMap`形式，但asMap视图与缓存交互需要注意：

- `cache.asMap()`包含当前所有加载到缓存的项。因此相应地，`cache.asMap().keySet()`包含当前所有已加载的键。
- `asMap().get(key)`实质上等同于`cache.getIfPresent(key)`，而且不会引起缓存项的加载。这和Map的语义约定一致。
- 所有读写操作都会重置相关缓存项的访问时间，包括`cache.asMap().get(key)`方法和`cache.asMap().put(K,V)`方法，但不包括`cache.asMap().containsKey(K)`方法，也不包括`cache.asMap()`的集合视图上的操作。比如，遍历`cache.asMap().entrySet()`不会重置缓存项的读取时间。

## 中断

缓存加载方法（如`Cache.get`）不会抛出`InterruptedException`。我们也可以让这些方法支持`InterruptedException`，但这种支持注定是不完备的。并且会增加所有使用者的成本，而且只有少数使用者实际获益。

`Cache.get`请求到未缓存的值时会遇到两种情况：当前线程加载值，或等待另一个正在加载值的线程。这两种情况下的中断是不一样的。等待另一个正在加载值的线程属于较简单的情况：使用可中断的等待就实现了中断支持；但当前线程加载值的情况就比较复杂了：因为加载值的`CacheLoader`是由用户提供的，如果它是可中断的，那我们也可以实现支持中断，否则我们也无能为力。

如果用户提供的`CacheLoader`是可中断的，为什么不让`Cache.get`也支持中断呢？从某种意义上来说，其实是支持的：如果`CacheLoader`抛出`InterruptedException`，`Cache.get`将l立刻返回（就和其他异常情况一样）；此外，在加载缓存值的线程中，`Cache.get`捕捉到`InterruptedException`后将恢复中断，而其他线程中的`InterruptedException`则被包装成了`ExecutionException`。

原则上，我们可以拆除包装，把`ExecutionException`变为`InterruptedException`，但这会让所有的`LoadingCache`使用者都要处理中断异常，即使它们提供的`CacheLoader`是不可中断的。如果你考虑到所有非加载线程的等待仍可以被中断，这种做法也许是值得的。但许多缓存只在单线程中使用，它们的用户仍然必须捕捉不可能抛出`InterruptedException`异常。即使是哪些跨线程共享缓存的用户，也只是有时候能中断它们的`get`调用，取决于那个线程先发出请求。

对于这个决定，我们的指导原则是让缓存始终表现得好像是在当前线程加载值。这个原则让使用缓存或每次都计算值可以简单地相互切换。如果老代码（加载值的代码）是不可中断的，那么新代码（使用缓存加载值的代码）多半也应该是不可中断的。

如上所述，Guava Cache在某种意义上支持中断。另一个意义上说，Guava Cache不支持中断，这使用`LoadingCache`成了一个有漏洞的抽象：当加载过程被中断了，就当做其他异常一样处理，这在大多数情况下是可以的；但如果多个线程在等待同一个缓存项，即使加载线程被中断了，它也不应该让其他线程都失败（捕获到包装在`ExecutionException`里的`InterruptedException`）正确的行为是让剩余的某个线程重试加载。为此，我们记录了一个bug。然而，与其冒风险修复这个bug，我们可能会花更多的时间去实现另一个建议`AsyncLoadingCache`，这个实现就会返回一个有正确中断行为的Future对象。









