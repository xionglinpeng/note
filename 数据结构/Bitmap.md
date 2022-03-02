# Bitmap

[TOC]

## 简介

bitmap是一个比特数组，即每一个数据元素都是一个比特位。每一个比特位表示的是真实数据的状态，而真实数据是该比特位位于比特数组的索引值。

## 原理

假设有一个整数集合{S}，其包含20亿个元素，现在需要找出元素k是存在否在集合S中。一种简单的方法是将这20亿个元素加载到内存中，然后依次遍历比较。这种方式带来的问题在于其耗时过长，时间复杂度O(n)，而另一个更严重的问题在于内存消耗。假设集合S中的元素类型为int型，一个int类型占4个字节，20亿个元素就约占7.45GB的内存，这是不可接受的。还有一种解决方案是通过磁盘对20亿数据分批次进行比较，不过此时O(n)的时间复杂度都算不了什么了，大部分时间将会消耗在磁盘IO上。

通过bitmap可以有效的解决该问题。根据需求可知，我们的目标是找出元素k是存在否在集合S中，假设`{S∈{0~40亿} | k<40亿}`，那么就意味着`[0~40亿]`中的每一个元素只有存在和不存在两种状态，我们通过一个比特位来表示——0表示不存在，1表示存在。而最大40亿的数据则需要40亿个比特位。40亿个比特位约等于0.466GB。因此，我们只需要0.466GB的空间就可以存储40亿个元素（当然，我们只有20亿个元素，不过由于是以元素作为索引，且数据范围在`0~40`亿之间，因此以40亿计算）。现在，我们只需要以元素k作为索引，找出该比特位为0还是为1，就可以判断其是否存在集合S中。时间复杂度O(1)。

## 实现

在Java语言中，一个int占4个字节，32位比特位。一个long占8个字节，64个比特位。

以long类型为例，现在声明一个long型数组：

```java
long[] words;
```

一个long占64个比特位，即意味着：

- words[0]可以表示0~63范围内的元素。
- words[1]可以表示64~127范围内的元素。
- words[2]可以表示128~191范围内的元素。
- ......

假设数组长度为N，给定任意整数k，**k/64**就可以得到M应该位于words的哪一个下标的long，**k%64**就可以知道其应该位于该long的哪一个比特位。

> int为32比特，long为64比特。设int或者long的大小为m比特（m = $2^n$）。由于m是2的N次幂，因此`k/m`和`k%m`转换为位运算操作：
>
> - `k/m = k >> n (m=2^n)`，如m=64，由于`64=2^6`，因此`k/64 = k >> 6`。
>
> - `k % m = k & (m - 1)`，如m=64，因此`k % 64 = k & (63)`
>
> 简化左移操作：
>
> 在Java语言中，左移的规则是丢弃最高位，低位补0。例如long型1L向左移的5位：`1L << 5`。高位的5个0将被丢弃，低位将补5个0。如果移动的位数超过该类型的最大位数，那么编译器会对移动的位数取模。例如long型的最大位数为64，则`1L << 88 = 1L << (88 % 64) = 1L << (88 & (64 - 1))`，即`1L << 88`实际上只是向左移动了24位。
> 公式：`1 << (k & (m - 1)) = 1 << k`

### 添加

给定元素88。88/64=1，因此88应该位于words[1]中。88&64=24，因此88应该位于words[1]的第24个比特位。

假设words[1]等于528,426，其二进制为`00000000 00001000 00010000 00101010`

由于88应该位于words[1]的第24个比特位，因此将1向左位移24位（1 << 24），然后与words[1]进行按位或（|）运算，就可以使words[1]的第24个比特位变为1。

计算过程如下所示：

```
528,426   00000000 00000000 00000000 00000000 00000000 00001000 00010000 00101010
1<<24     00000000 00000000 00000000 00000000 00000001 00000000 00000000 00000000
—————————————————————————————————————————————————————————————————————————————————
result    00000000 00000000 00000000 00000000 00000001 00001000 00010000 00101010
```

计算公式：`words[k/64] | (1 << (k & (64 - 1)))`

简化：`words[k >> 6] | (1 << k)`

### 删除

删除也是一样首先计算出给定元素的位置words[n]，然后进行位运算。

还是以88为例，首先将1向左位移24位，然后按位取反，最后再与words[1]进行按位与（&）运算。

计算过程如下所示：

```
1<<24        00000000 00000000 00000000 00000000 00000001 00000000 00000000 00000000
~(1<<24)     11111111 11111111 11111111 11111111 11111110 11111111 11111111 11111111
17,305,642   00000000 00000000 00000000 00000000 00000001 00001000 00010000 00101010
—————————————————————————————————————————————————————————————————————————————————————
result       00000000 00000000 00000000 00000000 00000000 00001000 00010000 00101010
```

计算公式：`words[k/64] & (~(1 << (k & (64 - 1))))`

简化：`words[k >> 6] & (~(1 << k))`

### 查找

首先将1向左位移24位，然后与words[1]进行按位与（&）运算。

计算过程如下所示：

```
1<<24        00000000 00000000 00000000 00000000 00000001 00000000 00000000 00000000
17,305,642   00000000 00000000 00000000 00000000 00000001 00001000 00010000 00101010
—————————————————————————————————————————————————————————————————————————————————————   
result       00000000 00000000 00000000 00000000 00000001 00000000 00000000 00000000
```

如果等于0，则表示不存在，如果不等于，则表示存在。

计算公式：`words[k/64] & (1 << (k & (64 - 1)))`

简化：`words[k >> 6] & (1 << k)`

## 应用

排序

应用bitmap排序是基于桶排序的思想。标准的桶排序，计数排序以及基数排序由于辅助内存空间的限制，其只能针对最大和最小元素标准方差较小的可转换为整数元素的集合进行排序。而bitmap可以针对超大范围的集合进行排序，但是该集合不能存储重复元素，否则就不适合。

排重

包含N个元素的整数集合S，在其对应的比特位上，如果元素存在就将其设置为1，如果不存在则为0。去重时，只需要操作元素k在其对应的比特位是否为1，则就可知道该元素是否已经存在。

查找

查找元素k是否存在，只需要操作元素k在其对应的比特位是否为1，则就可知道该元素是否已经存在。

## 分析

优点

- 运算效率高，不需要进行比较和位移。
- 占用内存少，例如0~10,000,000共1千万个数据，只需要占用10,000,000/8 = 1,250,000Byte = 1.2MB

缺点

- 数据不可重复，即不能对包含重复元素的集合进行排除和查找。
- 只有当数据比较密集（数据量比较多）时才有优势。例如集合S只有元素1和20亿两个元素，那就不太合适。

## 扩展

1个比特位可以表示两种状态，如果是2个比特位的话可以表示三种状态，3个比特位可以表示8种状态，以此类推。如果数据需要表示2种以上状态时，可以使用多个比特位来表示。其原理和操作与1个比特位本质是一样的。

## BitSet

### 简介

JDK中，`java.util.BitSet`是对bitmap的实现。这个类实现了一个根据需要增长的位向量。位集的每个组成部分都有一个boolean值。BitSet的位由非负整数索引。可以检查、设置或清除单个索引位。一个BitSet可以通过逻辑与(&)、逻辑或(|)和逻辑异或(^)操作来修改另一个BitSet的内容。

默认情况下，集合中的所有位初始值为false。

每个位集都有一个当前大小，即位集当前使用的空间位数。注意，大小与位集的实现有关，所以它可能会随着实现而改变。位集的长度与位集的逻辑长度相关，它的定义与实现无关。

除非另有说明，否则在BitSet中向任何方法传递空参数都会导致`NullPointerException`。

BitSet对于没有外部同步的多线程使用是不安全的。

### 源码

```java
public class BitSet implements Cloneable, java.io.Serializable {
    /*
     * BitSets被打包成由"words"组成的数组。当前一个word
     * 是一个long型。由64位组成，需要6个比特位。
     * word大小的选择纯粹是由性能考虑决定的。
     */
    private final static int ADDRESS_BITS_PER_WORD = 6;
    private final static int BITS_PER_WORD = 1 << ADDRESS_BITS_PER_WORD;
    private final static int BIT_INDEX_MASK = BITS_PER_WORD - 1;

    /* Used to shift left or right for a partial word mask */
    private static final long WORD_MASK = 0xffffffffffffffffL;

    /**
     * @serialField bits long[]
     *
     * BitSet中的bits。第i位存储在bits[i/64]里位于偏移量
     * i % 64的比特位中(其中位0表示最低有效位，63表示最高有效位)。
     */
    private static final ObjectStreamField[] serialPersistentFields = {
        new ObjectStreamField("bits", long[].class),
    };

    /**
     * 内部比特数组字段
     */
    private long[] words;

    /**
     * “words”数组的大小
     */
    private transient int wordsInUse = 0;

    /**
     * “words”大小是否由用户指定。
     */
    private transient boolean sizeIsSticky = false;
    
    /**
     * 给定一个bit索引，返回包含它的word索引。
     */
    private static int wordIndex(int bitIndex) {
        return bitIndex >> ADDRESS_BITS_PER_WORD;
    }
    ......
}
```

添加

```java
public void set(int bitIndex) {
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);
	//计算位于word[]索引 ： bitIndex >> 6
    int wordIndex = wordIndex(bitIndex);
    expandTo(wordIndex);
    // words[wordIndex] = words[k >> 6] | (1 << k)
    words[wordIndex] |= (1L << bitIndex); // Restores invariants
    checkInvariants();
}
```

查找

```java
public boolean get(int bitIndex) {
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);
    checkInvariants();
    int wordIndex = wordIndex(bitIndex);
    //(words[k >> 6] & (1 << k)) != 0
    return (wordIndex < wordsInUse)
        && ((words[wordIndex] & (1L << bitIndex)) != 0);
}
```

删除

```java
public void clear(int bitIndex) {
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);
    int wordIndex = wordIndex(bitIndex);
    if (wordIndex >= wordsInUse)
        return;
    //words[wordIndex] = words[k >> 6] & (~(1 << k))
    words[wordIndex] &= ~(1L << bitIndex);
    recalculateWordsInUse();
    checkInvariants();
}
```

### API

#### 构造函数创建BitSet

1. `BitSet()`
   创建一个新的位集。
2. `BitSet(int nbits)`
   创建一个位集合，其初始大小足够大以显式表示具有 0到 nbits-1范围内的索引的位。

#### 静态函数valueOf创建BitSet

1. `static BitSet valueOf(byte[] bytes)`
   返回包含给定字节数组中所有位的新位集合。

2. `static BitSet valueOf(ByteBuffer bb)`
   返回一个新的位集，其中包含给定字节缓冲区中位置和极限之间的所有位。

3. `static BitSet valueOf(long[] longs)`
   返回包含给定长数组中所有位的新位集。

4. `static BitSet valueOf(LongBuffer lb)`
   返回包含给定长缓冲区中其位置和极限之间的所有位的新位集合。

#### BitSet位运算函数

1. `void and(BitSet set)`
   执行此参数位置位的此目标位设置的逻辑 AND 。
2. `void or(BitSet set)`
   使用位设置参数执行该位的逻辑 或 。
3. `void xor(BitSet set)`
   使用位设置参数执行该位的逻辑 异或 。
4. `void andNot(BitSet set)`
   清除该BitSet中所有对应位在指定BitSet中被设置的位。

#### 添加元素

1. `void set(int bitIndex)`
   将指定索引处的位设置为 true 。
2. `void set(int bitIndex, boolean value)`
   将指定索引处的位设置为指定值。
3. `void set(int fromIndex, int toIndex)`
   将指定的 fromIndex (包含)到指定的 toIndex(排除)的位设置为true 。
4. `void set(int fromIndex, int toIndex, boolean value)`
   将指定的 fromIndex (包含)到指定的 toIndex(排除)的位设置为指定值。

#### 获取元素

1. `boolean get(int bitIndex)`
   返回具有指定索引的位的值。
2. `BitSet get(int fromIndex, int toIndex)`
   返回一个新 BitSet组成位从这个 BitSet从 fromIndex(包含)至 toIndex (排除)。

#### 清理元素

1. `void clear()`
   将此BitSet中的所有位设置为 false 。
2. `void clear(int bitIndex)`
   将索引指定的位设置为 false 。
3. `void clear(int fromIndex, int toIndex)`
   将指定的 fromIndex (包含)到toIndex(排除)的位设置为  false 。

#### Flip

1. `void flip(int bitIndex)`
   将指定索引处的位设置为其当前值的补数。
2. `void flip(int fromIndex, int toIndex)`
   将从指定的fromIndex(包含)到指定的toIndex(排除)的每一位设置为其当前值的补数。

#### Next and Previous

1. `int nextClearBit(int fromIndex)`
   返回设置为false的第一个位的索引，该索引在指定的起始索引时或之后出现。

2. `int nextSetBit(int fromIndex)`

   返回设置为true的第一个位的索引，该索引在指定的起始索引时或之后出现。如果不存在这样的位，则返回-1。

   要迭代BitSet中的true位，使用下面的循环:

   ```java
   for (int i = bs.nextSetBit(0); i >= 0; i = bs.nextSetBit(i+1)) {
        // operate on index i here
        if (i == Integer.MAX_VALUE) {
            break; // or (i+1) would overflow
        }
    }
   ```

3. `int previousClearBit(int fromIndex)`
   返回设置为false的最近位的索引，该索引在指定的起始索引上或之前出现。如果不存在这样的位，或者-1作为起始索引，则返回-1。

4. `int previousSetBit(int fromIndex)`
   返回设置为true的最近位的索引，该索引在指定的起始索引上或之前出现。如果不存在这样的位，或者-1作为起始索引，则返回-1。

   要迭代BitSet中的true位，使用下面的循环:

   ```java
   for (int i = bs.length(); (i = bs.previousSetBit(i-1)) >= 0; ) {
        // operate on index i here
    }
   ```

#### 其他

1. `int cardinality()`
   返回此 BitSet设置为 true的 BitSet 。
2. `Object clone()`
   克隆这个 BitSet产生一个新的 BitSet等于它。
3. `boolean equals(Object obj)`
   将此对象与指定对象进行比较。
4. `int hashCode()`
   返回此位集的哈希码值。
5. `boolean intersects(BitSet set)`
   如果指定，则返回true BitSet具有设置为任何位 true这也被设置为 true这个 BitSet 。
6. `boolean isEmpty()`
   如果此 BitSet包含设置为 true位，则返回true。
7. `int ()`
   返回这个BitSet的“逻辑大小”： BitSet加上最高位的索引。
8. `int size()`
   返回此 BitSet实际使用的空间位数，以表示位值。
9. `IntStream stream()`
   返回此 BitSet包含设置状态位的索引流。
10. `byte[] toByteArray()`
    返回一个包含该位集中所有位的新字节数组。
11. `long[] toLongArray()`
    返回一个包含该位集合中所有位的新长数组。
12. `String toString()`
    返回此位集的字符串表示形式。