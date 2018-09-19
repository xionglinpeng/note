[TOC]


# HashMap源码分析

HashMap是基于数组和链表实现的


首先我们看看HashMap的成员变量，HashMap有6个静态常量

```
    /**
     * 默认的初始容量-必须是2的幂。
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * 最大容量，如果较高的值是由任何带有参数的构造函数隐式指定的。
     * 必须是2的幂 <= 1<<30。
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 在构造函数中未指定时使用的负载因子。
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 使用树而不是列出一个bin的bin计数阈值。
     * 当向具有至少这么多节点的bin中添加元素时，bin被转换为树。
     * 这个值必须大于2，并且应该至少为8，以便与树删除中关于收缩后转换回普通容器的假设相匹配。
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 在内存操作期间取消(分割)存储库的存储计数阈值。
     * 应该小于TREEIFY_THRESHOLD，
     * 并且最多6个以网孔的收缩检测在去除下。
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 最小的表容量，可为容器进行树状排列。
     * (否则，如果在一个bin中有太多节点，表将被调整大小。)
     * 应该至少是4 * TREEIFY_THRESHOLD以避免冲突
     * 在调整大小和树化阈值之间。
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    
    
    
    /* ---------------- Fields -------------- */

    /**
     * 表，在第一次使用时初始化，并根据需要调整大小。
     * 分配时，长度总是2的幂。
     * (我们还允许某些操作中的长度为零，以允许目前不需要的引导机制。)
     * 
     */
    transient Node<K,V>[] table;

    /**
     * 保存缓存entrySet()。注意，AbstractMap字段用于keySet()和values()。
     * 
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * 这个映射中包含的键值映射的数量。
     */
    transient int size;

    /**
     * 这个HashMap被结构上修改的次数结构修改是那些改变HashMap中的映射数量或以其他方式修改其内部结构(例如，rehash)的次数。
     *
     * 这个字段用于使HashMap的集合视图上的迭代器快速失败。
     * (见ConcurrentModificationException)。  
     */
    transient int modCount;

    /**
     * 调整大小的下一个大小值(容量*负载因子)。
     *
     * @serial
     */
    // (序列化时javadoc描述为真。
    // 另外，如果表数组没有被分配，这个字段保持初始数组容量，
    // 或者0表示DEFAULT_INITIAL_CAPACITY。)
    // 
    int threshold;

    /**
     * 哈希表的负载因子。
     *
     * @serial
     */
    final float loadFactor;
```

HashMap数组默认容量大小，2的4次幂，即16（注意：这里设置为1 << 4是有原因的，后面再具体描述）。
```
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

HashMap数组的最大容量，2的30次幂
```
static final int MAXIMUM_CAPACITY = 1 << 30;
```

默认加载因子
```
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```


判断是否需要将链表转换为树的阈值
```
static final int TREEIFY_THRESHOLD = 8;
```

存储数据的Node节点数组表

```
transient Node<K,V>[] table;
```

HashMap存放数量的大小
```
transient int size;
```

这个属性翻译为“阈值”，即HashMap进行扩容的阈值（loadFactor * capacity）,但是在初始化时是用于作为容量的大小
```
int threshold;
```

负载因子，计算扩容阈值的比例

```
final float loadFactor;
```



我们知道HashMap是由数组和链表构成的，那么它的每一个节点是如何构成的呢，如下静态内部类Node<K,V>

定义了4个属性
- hash : key的hash值
- key : 键
- value : 值
- next : 链表的下一个节点

没有上一个节点属性，可以看出是一个单链表

注意这里的`setValue()`方法，是设置新值返回旧值

```
/**
 * 基本哈希bin节点，用于大多数条目。
 * (参见下面的TreeNode子类，以及LinkedHashMap中的条目子类)
 */
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

# put方法


```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //判断桶是否为空，为空就要进行初始化（resize()中会判断进行初始化）。
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //取模运算定位key在当前桶中的位置，并判断当前位置是否为空，为空即表示没有hash冲突，就直接创建一个新的节点保存上去 
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
    //既不需要初始化，并且hash冲突了，则表示当前节点是一个链表节点，进行链表的判断操作
        Node<K,V> e; K k;
        //当前桶有值（hash冲突），就比较它们的key是否相等，相等就赋值给局部变量e
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //当前节点是否是红黑树，是就要按照红黑树的方式写入数据    
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //依次迭代循环找到链表每一个节点
            for (int binCount = 0; ; ++binCount) {
                //如果找到最后一个节点后面没有节点了，就直接创建一个新的节点保存上去
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //链表长度是否大于阈值，大于就转换成红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //如果遍历过程中找到的key相等，就退出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //e != null表示存在相同的key，将旧值覆盖，并且返回旧值
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            //LinkedHashMap扩展实现
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //键值对添加成功，数量+1，并判断节点数量是否大于扩容阈值，大于则扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
# get方法

```
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //桶不为空，则并且当前key Hash取模定位桶位置，其有不为空，若为空，这直接返回null
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //比较key是否相等，相等，则返回对应的值
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //链表的下一个节点不为空
        if ((e = first.next) != null) {
            //如果是红黑树，则按照红黑树的方式取值
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //迭代链表每一个节点，直到找到相等的节点，返回对应的值，或者全查找完
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

# 扩容

```
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    //桶的容量大于0
    if (oldCap > 0) {
        //如果桶的容量大于最大容量，则设置扩容阈值为Integer.MAX_VALUE，并直接返回
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //扩容为2次幂（2倍）。如果扩容后的小于最大容量，并且扩容前的容量大于默认容量（16），则将扩容阈值也扩容为2次幂
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    //如果旧扩容阈值大于0，则设置初始容量
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    //否则，初始容量为默认容量，扩容阈值为默认容量*默认加载因子    
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //新扩容阈值等于0，则设置新扩容阈值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //将新的扩容阈值保存到成员变量threshold
    threshold = newThr;
    //创建一个新的Node节点数组
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    //将新的Node节点数组保存到桶table
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

# hash运算

进行hash运算的源码如下：首先判断key是否为null，如果为null，则直接返回0，如果不为null，则计算器hashCode，并且与其低十六位异或运算。
```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
那么为什么要这么做呢？

首先看看其源码注释的说明

计算key.hashCode()，并将(XORs)的高比特位分散到低比特位。
因为这个表使用的是2次方掩蔽，
仅在当前掩码以上的位上变化的哈希组合总是会发生碰撞。
(已知的例子中有一组浮点键，它们在小表中保存连续的整数。)
因此，我们应用了一种转换，将高比特的影响向下传播。
比特传播的速度、效用和质量之间存在权衡。
因为许多常见的哈希集已经合理地分布了
(所以不要从传播中获益)
因为我们用树来处理垃圾箱里的大量碰撞，
我们只是用最便宜的方法来减少系统损失，
此外，由于表边界的关系，最高位的影响在索引计算中是永远不会用到的。


# 取模运算

在我们阅读put方法源码的时候发现，有一段代码：`(n - 1) & hash`，其是通过key的hash值与表的长度减1的值进行位与运算来的得到其位置的，换算一下：
```
key.hash & (table.length - 1)
```
那么为什么要通过这种方式来定位key的位置呢？

在说明为什么之前，我们先给出一个结论，然后再去证明：

结论如下：
1. HashMap是通过key的hash值与表的长度取模定位key的位置的。
2. `m % o == m & (o - 1)` (o为2的n次幂，在这里大家可能不理解为什么o为2的n次幂，在后面会有说明，暂且先记住o为2的n次幂即可)

先总结一下，然后在去证明为什么`m % o == m & (o - 1)`。

通过上面的两条结论可以知道：
- `(n - 1) & hash`是在取模，而位与运算的效率是比`%`取模的方式效率更高。
- 余数值必定小于被除数，在这里表的长度就是被除数，取模得出值就必定在表的索引范围之内。而之所以减1，是因为数组的下标是从0开始的。
- 例如 ：9 % 4 = 1，1001(9) & 0011(4-1) = 0001(1) ，`m % o == m & (o - 1)`成立。
- 例如 ：9 % 6 = 3，1001(9) & 0101(6-1) = 0001(1) ，`m % o == m & (o - 1)`不成立。
- 
证明：`m % o == m & (o - 1)`

前提条件：
- o为2的n次幂；例如:

```math
2^0/2^1/2^2/2^3/.....  
```

- 所有2的n次幂整数的二进制位1(0n)，0n表示n个0；例如:

```math
2^2 = ...0000100

2^3 = ...0001000
```
- 所有（2的n次幂整数-1）的二进制位01n，1n表示n个1；例如:

```math
2^2-1 = 3 = ...0000011

2^3-1 = 7 = ...0000111
```


任意十进制数可以转换为以下二进制表示的公式
```math
B_nB_{n-1}B_{n-2}B_{n-3}...B_2B_1B_0 = B_n*2^n + B_{n-1}*2^{n-1} + B_{n-2}*2^{n-2} + B_{n-3}*2^{n-3} + ... + B_2*2^2 + B_1*2^1 + B_0*2^0


```
除法运算规则

```math
(a+b)\div c = (a\div c)+(b\div c) = \frac{a}{c}+\frac{b}{c}
```
所以又有如下表示
```math
(B_nB_{n-1}B_{n-2}B_{n-3}...B_2B_1B_0)\div2^k = \frac{B_n*2^n}{2^k} + \frac{B_{n-1}*2^{n-1}}{2^k} + \frac{B_{n-2}*2^{n-2}}{2^k} + \frac{B_{n-3}*2^{n-3}}{2^k} + ... + \frac{B_2*2^2}{2^k} + \frac{B_1*2^1}{2^k} + \frac{B_0*2^0}{2^k}
```



Bit位都为1或0，所以

当 0<=k<=n 时，余数为

```math
B_{k-1}*2^{k-1} + B_{k-2}*2^{k-2} + B_{k-3}*2^{k-3} + ... + B_2*2^2 + B_1*2^1 + B_0*2^0
```

当 k > n，余数为整个十进制整数。

假设

```math
58\div 2^2 = 14...2
```

```math
111010/2^2 = \frac{111010}{2^2}
```

```math
(2^5 + 2^4 + 2^3 + 2^0 + 2^1 + 2^0)/2^2 = \frac{2^5}{2^2} + \frac{2^4}{2^2} + \frac{2^3}{2^2} + \frac{2^0}{2^2} + \frac{2^1}{2^2} + \frac{2^0}{2^2} = \frac{2^5}{2^2} + \frac{2^4}{2^2} + \frac{2^3}{2^2} + \frac{2^1}{2^2}
```

```math
(100000 + 10000 + 1000 + 0 + 10 + 0)/100 = \frac{100000}{100} + \frac{10000}{100} + \frac{1000}{100} + \frac{0}{100} + \frac{10}{100} + \frac{0}{100} = \frac{100000}{100} + \frac{10000}{100} + \frac{1000}{100} + \frac{10}{100}
```

```math
\frac{...000111010}{...000000100}
```

```math
\frac{...000111010}{2^2-1} = \frac{...000111010}{...000000011} = ...000000010
```


```math

```

# 扩容重定位运算


```math
\frac{...000111010}{...000000100}
```

```math
\frac{...000111010}{2^3} = \frac{...000111010}{...000001000}
```
```math
\frac{...000111010}{2^3-1} = \frac{...000111010}{...000000111} = ...000000010
```


```math
(100000 + 10000 + 1000 + 0 + 10 + 0)/100 = \frac{100000}{1000} + \frac{10000}{1000} + \frac{1000}{1000} + \frac{0}{1000} + \frac{10}{1000} + \frac{0}{1000} = \frac{100000}{1000} + \frac{10000}{1000} + \frac{1000}{1000} + \frac{10}{1000}
```


```math
(100000 + 10000 + 1000 + 100 + 10 + 0)/100 = \frac{100000}{100} + \frac{10000}{100} + \frac{1000}{100} + \frac{100}{100} + \frac{10}{100} + \frac{0}{100} = \frac{100000}{100} + \frac{10000}{100} + \frac{1000}{100} + \frac{100}{100}+ \frac{10}{100}
```
```math
(100000 + 10000 + 1000 + 100 + 10 + 0)/100 = \frac{100000}{1000} + \frac{10000}{1000} + \frac{1000}{1000} + \frac{100}{1000} + \frac{10}{1000} + \frac{0}{1000} = \frac{100000}{1000} + \frac{10000}{1000} + \frac{1000}{1000} + \frac{100}{1000}+ \frac{10}{1000}
```


所以如果

```math
E = mc^2
```


，如果被除数增加二次幂，那么可以通过原被除

# 线程安全性