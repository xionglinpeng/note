# bitmap



## setbit

```shell
SETBIT key offset value
```

> 起始版本：2.2.0
>
> 时间复杂度：O(1)

设置或者清空key的value（字符串）在offset处的bit值。

那个位置的bit要么被设置，要么被清空，这个由value（只能是0或者1）来决定。当key不存在的时候，就创建一个新的字符串value。要确保这个字符串大到在offset处有bit值。参数offset需要大于等于0，并且小于232（限制bitmap大小为512）。当key对应的字符串增大的时候，新增的部分bit值都是设置为0。

警告：当set最后一个bit（offset等于2^{32}-1）并且key还没有一个字符串value或者其value是个比较小的字符串时，Redis需要立即分配所有内存，这有可能会导致服务阻塞一会。

在一台2.10MacBook Pro上：

- offset为2^{32}-1（分配512MB）需要~300ms；
- offset为2^{30}-1（分配128MB）需要~80ms;
- offset为2^{28}-1（分配32MB）需要~30ms;
- offset为2^{26}-1（分配8MB）需要~8ms;

注意：一旦第一次内存分配完，后面对同一个key调用`setbit`就不会预先得到内存分配。

### 返回值

在offset处原来的bit值

### Command Operation

```shell
127.0.0.1:6379> setbit hello 1 1
(integer) 0
127.0.0.1:6379> setbit hello 2 1
(integer) 0
127.0.0.1:6379> setbit hello 4 1
(integer) 0
127.0.0.1:6379> setbit hello 9 1
(integer) 0
127.0.0.1:6379> setbit hello 10 1
(integer) 0
127.0.0.1:6379> setbit hello 13 1
(integer) 0
127.0.0.1:6379> setbit hello 15 1
(integer) 0
127.0.0.1:6379> get hello
"he"
```

### Code Operation

```java
/**
  * 模拟一年365天用户签到记录，1表示签到，0表示未签到。
  */
private void setbit(){
    ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();
    Random random = new Random();
    for (int i = 0; i < 365; i++) {
        boolean b = random.nextBoolean();
        operations.setBit("bitmap",i, b);
        System.out.print(b?1:0);
    }
    System.out.println();
}
```

输出

```
01010100001100101000010100100111110101001111110110000110001000101111110000110100100101010001111010101001001101111100111110011111101110011101000011101001100010101000111011100010011100001110010010110001011010000100011110100111000101000100101001111101011000111110000001001100000000001000100001100110100100011100001010010100111110100000011101111111011100011100111000001
```

## getbit

```shell
GETBIT key offset
```

> 起始版本：2.2.0
>
> 时间复杂度：O(1)

返回key对应的string在offset处的bit值，当offset超出了字符串长度的时候，这个字符串就被假定为由0比特填充的连续空间。当key不存在的时候，它就认为是一个空字符串。所以offset总是超出范围，然后value也被认为是由0比特填充的连续空间到内存分配。

### 返回值

在offset处的bit值。

### Command Operation

```shell
127.0.0.1:6379> getbit hello 0
(integer) 0
127.0.0.1:6379> getbit hello 1
(integer) 1
127.0.0.1:6379> getbit hello 2
(integer) 1
127.0.0.1:6379> getbit hello 3
(integer) 0
127.0.0.1:6379> getbit hello 4
(integer) 1
127.0.0.1:6379> getbit hello 5
(integer) 0
```



## bitcount

```shell
$ bitcount key [start end]
```



## bitpos





## bitfiled

> 其实版本：2.6.0
>
> 时间复杂度：O(N)

统计字符串被设置为1的bit数。

一般情况下，给定的整个字符串都会被进行计数，通过指定额外的start或end参数，可以让计数只在特定的位上进行。

start和end参数的设置和`getrange`命令类似，都可以使用负数值；比如-1表示最后一个为，而-2表示倒数第二个为，依次类推。

不存在的key被当成是空字符串类处理，因此对一个不存在的key进行`bitcount`操作，结果为0。

**返回值**

被设置为1的位的数量。

**例子**



模式：使用bitmap实现用户上线次数统计

bitmap对于一些特点类型的计算非常有效。

假设现在我们希望记录自己网站上的用户的上线评论，比如说，计算用户A上线了多少天，诸如此类，以此作为数据。从而决定让那些用户参数beta测试等活动——这个模式可以使用`setbit`和`bitcount`来实现。比如说，每当用户在某一天上线的时候，我们就使用`setbit`，以用户名作为key，将那天所代表的网站的上线日作为`offset`参数，并将这个`offset`上的位设置为1。

举个例子，如果今天是网站上线的第100天，而用户peter在今天阅览过网站，那么执行命令`setbit perter 100 1`；如果明天peter也继续阅览网站，那么执行命令`setbit peter 101 1`，以此类推。当要计算peter总共以来的上线次数时，就是使用`bitcount`命令；执行`bitcount peter`，得出的结果就是peter上线的总天数。

性能

前面的上线次数统计例子，即使运行10年，占用的空间也只是每个用户10*365比特位（bit），也即是每个用户456个字节。对于这种大小的数据来说，`bitcount`的处理速度就像`get`和`incr`这种O(1)复杂度的操作一样快。

如果你的bitmap数据非常大，那么可以考虑使用一下两种方法：

- 将一个大的bitmap分散到不同的key中，作为小的bitmap来处理。使用Lua脚本可以很方便的完成这一工作。
- 使用`bitcount`的start和end参数，每次只对所需的部分位进行计算，将位的积累工作（accumulating）放到客户端进行。并且对结果进行缓存（caching）。



## bitop

```shell
$ bitop opeartion destkey key [key]
```

opeartion可以是`AND`、`OR`、`XOR`、`NOT`这四种操作中的任意一种：

- `bitop AND destkey key [key...]`，对一个或多个key求逻辑并，并将结果保存到`destkey`。
- `bitop OR destkey key [key...]`，对一个或多个key求逻辑或，并将结果保存到`destkey`。
- `bitop XORdestkey key [key...]`，对一个或多个key求逻辑异或，并将结果保存到`destkey`。
- `bitop NOT destkey key`，对给定的key求逻辑非，并将结果保存到`destkey`。

> 除了NOT操作之外，其他操作都可以结果一个或多个key作为输入。

当bitop处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作0。

空的key也被看作是包含0的字符串序列。

**可用版本：**

	=2.6.0

**时间复杂度：**

	O(N)

**返回值：**

	保存到destkey的字符串的长度，和输入key中最长的字符串的长度相等。

**性能：**

`BITOP`可能是一个缓慢的命令，它的时间复杂度是O(N)。 在处理长字符串时应注意一下效率问题。

对于实时的指标和统计，涉及大输入一个很好的方法是 使用bit-wise操作以避免阻塞主实例。



```
 CROSSSLOT Keys in request don't hash to the same slot
```

### Command Opeartion

下面是Redis-Cli命令行界面操作示例

设置了`{m}a=110`，`{m}b=011`，逻辑并运算，destkey为`{m}c`，`{m}c=010`。

```shell
[root@localhost ~]# redis-cli
127.0.0.1:6379> setbit {m}a 0 1
(integer) 0
192.168.56.5:6379> setbit {m}a 1 1
(integer) 0
192.168.56.5:6379> setbit {m}a 2 0
(integer) 0
192.168.56.5:6379> setbit {m}b 0 0
(integer) 0
192.168.56.5:6379> setbit {m}b 1 1
(integer) 0
192.168.56.5:6379> setbit {m}b 2 1
(integer) 0
192.168.56.5:6379> bitop and {m}c {m}a {m}b
(integer) 1
192.168.56.5:6379> getbit {m}c 0
(integer) 0
192.168.56.5:6379> getbit {m}c 1
(integer) 1
192.168.56.5:6379> getbit {m}c 2
(integer) 0
```

### Code Operation

下面是使用`Spring-Data-Redis`API编写的代码示例：

```java
private void bitop(){
    ValueOperations<Object, Object> operations = redisTemplate.opsForValue();
    //逻辑并
    RedisStringCommands.BitOperation and = RedisStringCommands.BitOperation.AND;
    //逻辑或
    RedisStringCommands.BitOperation not = RedisStringCommands.BitOperation.NOT;
    //逻辑异或
    RedisStringCommands.BitOperation or = RedisStringCommands.BitOperation.OR;
    //逻辑非
    RedisStringCommands.BitOperation xor = RedisStringCommands.BitOperation.XOR;
    //设置{o}:a = 0101
    operations.setBit("{bitop}:a",0,false);
    operations.setBit("{bitop}:a",1,true);
    operations.setBit("{bitop}:a",2,false);
    operations.setBit("{bitop}:a",3,true);
    //设置{o}:b = 0110
    operations.setBit("{bitop}:b",0,false);
    operations.setBit("{bitop}:b",1,true);
    operations.setBit("{bitop}:b",2,true);
    operations.setBit("{bitop}:b",3,false);

    System.out.println("bitOp:A = 0101");
    System.out.println("bitOp:B = 0110");
    //用于输出{bitop}:c
    Supplier<String> printBitOpC = ()->{
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < 4; i++) {
            Boolean bit = operations.getBit("{bitop}:c", i);
            stringBuilder.append(bit?1:0);
        }
        return stringBuilder.toString();
    };
    //逻辑运算（不包括非）
    Consumer<RedisStringCommands.BitOperation> bitOp = operation-> {
        Long execute = stringRedisTemplate.execute((RedisCallback<Long>) connection ->
                                                   connection.bitOp(operation, "{bitop}:c".getBytes(), "{bitop}:a".getBytes(), "{bitop}:b".getBytes()));
        System.out.print("实际结果："+printBitOpC.get()+"  ");
        System.out.println("execute = "+execute);
    };
    //逻辑运算：非
    Consumer<String> bitOpNot = key-> {
        Long execute = stringRedisTemplate.execute((RedisCallback<Long>) connection ->
                                                   connection.bitOp(not, "{bitop}:c".getBytes(), key.getBytes()));
        System.out.print("实际结果："+printBitOpC.get()+"  ");
        System.out.println("execute = "+execute);
    };
    //逻辑并运算
    System.out.print("AND:预期结果：0100,  ");
    bitOp.accept(and);
    //逻辑或运算
    System.out.print("OR :预期结果：0111,  ");
    bitOp.accept(or);
    //逻辑异或运算
    System.out.print("XOR:预期结果：0011,  ");
    bitOp.accept(xor);
    //逻辑非运算
    System.out.print("NOT:{bitop}:a->预期结果：1010,  ");
    bitOpNot.accept("{bitop}:a");
    System.out.print("NOT:{bitop}:b->预期结果：1001,  ");
    bitOpNot.accept("{bitop}:b");
}
```

输出结果：

```
bitOp:A = 0101
bitOp:B = 0110
AND:预期结果：0100,  实际结果：0100  execute = 1
OR :预期结果：0111,  实际结果：0111  execute = 1
XOR:预期结果：0011,  实际结果：0011  execute = 1
NOT:{bitop}:a->预期结果：1010,  实际结果：1010  execute = 1
NOT:{bitop}:b->预期结果：1001,  实际结果：1001  execute = 1
```

