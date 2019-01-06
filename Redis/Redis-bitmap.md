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

### Code Operation

```java
private void getbit(){
    ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();
    for (int i = 0; i < 365; i++) {
        Boolean bitmap = operations.getBit("bitmap", i);
        assert bitmap != null;
        System.out.print(bitmap?1:0);
    }
    System.out.println();
}
```

输出结果：

```
00000010110000101110110101111000100000110111101011111001001101000111010001001000000000101000110011001011001111110000001010110110001001101001011110011000101100011000010100001100011100111011011110100010101101100000000001000011110010101110000000010001100000100000011111110110111011011110001000110111011010001100101001000111101010000000000110010001111000000001100101010
```

## bitcount

```shell
$ bitcount key [start end]
```

> 起始版本：2.6.0
时间复杂度：O(N)

统计字符串被设置为1的bit数。

 一般情况下，给定的整个字符串都会被进行计数，通过指定额外的start或end参数，可以让计数只在特定的位上进行。

 start和end参数的设置和`getrange`命令类似，都可以使用负数值；比如-1表示最后一个，而-2表示倒数第二个为，依次类推。

不存在的key被当成是空字符串类处理，因此对一个不存在的key进行`bitcount`操作，结果为0。

 ### 返回值

 被设置为1的位的数量。

### Command Operation

```shell
127.0.0.1:6379> bitcount w
(integer) 2
127.0.0.1:6379> bitcount w 2 6
(integer) 0
127.0.0.1:6379> bitcount w -1 -5
(integer) 0
127.0.0.1:6379> bitcount n
(integer) 0
```

模式：使用bitmap实现用户上线次数统计

bitmap对于一些特点类型的计算非常有效。

假设现在我们希望记录自己网站上的用户的上线评论，比如说，计算用户A上线了多少天，诸如此类，以此作为数据。从而决定让那些用户参数beta测试等活动——这个模式可以使用`setbit`和`bitcount`来实现。比如说，每当用户在某一天上线的时候，我们就使用`setbit`，以用户名作为key，将那天所代表的网站的上线日作为`offset`参数，并将这个`offset`上的位设置为1。

举个例子，如果今天是网站上线的第100天，而用户peter在今天阅览过网站，那么执行命令`setbit perter 100 1`；如果明天peter也继续阅览网站，那么执行命令`setbit peter 101 1`，以此类推。当要计算peter总共以来的上线次数时，就是使用`bitcount`命令；执行`bitcount peter`，得出的结果就是peter上线的总天数。

**性能**

前面的上线次数统计例子，即使运行10年，占用的空间也只是每个用户10*365比特位（bit），也即是每个用户456个字节。对于这种大小的数据来说，`bitcount`的处理速度就像`get`和`incr`这种O(1)复杂度的操作一样快。

如果你的bitmap数据非常大，那么可以考虑使用一下两种方法：

- 将一个大的bitmap分散到不同的key中，作为小的bitmap来处理。使用Lua脚本可以很方便的完成这一工作。
- 使用`bitcount`的start和end参数，每次只对所需的部分位进行计算，将位的积累工作（accumulating）放到客户端进行。并且对结果进行缓存（caching）。

### Code Operation

```java
/**
  * 查询今年全部的签到天数
  */
private void bitCount1(){
    Long days = stringRedisTemplate.execute((RedisCallback<Long>) connection -> connection.bitCount(stringRedisTemplate.getStringSerializer().serialize("bitmap")));
    System.out.printf("今天年签到天数：%d天。\n", days);
}

/**
  * 查询第96天到200天的签到天数。
  * 注意：start和end是以字节为单位
  */
private void bitCount2(){
    Long execute = stringRedisTemplate.execute((RedisCallback<Long>) connection ->
 connection.bitCount(
     stringRedisTemplate.getStringSerializer().serialize("bitmap"),96 / 8, 200 / 8));
    System.out.printf("今年第96天到200天签到%d天。\n", execute);
}
```

输出结果：

```
今天年签到天数：202天。
今年第96天到200天签到63天。
```

## bitpos

```shell
BITPOS key bit [start] [end]
```

> 起始版本：2.8.7
>
> 时间复杂度：O(N)

返回字符串里面第一个被设置为1或者0的bit位。

返回一个文职，那字符串当做一个从左到右的字节数组，第一个符合条件的在位置0，其次在位置8，等等

`getbit`和`setbit`也是相似操作字节位的命令。

默认情况下整个字符串都会被检索一次，只有在指定start和end参数（指定的start和end位是可行的），该范围被解释为一个字节的范围，而不是一系列的位。所以start=0并且end=2是指前三个字节范围内查找。

注意，返回的位的位置始终是从0开始的，即使使用了start来指定了一个开始字节也是这样。

和`getrange`命令一样，start和end也可以包含负值，负值将从字符串的末尾开始计算，-1是字符串的最后一个字节，-2是倒数第二个，等等。

不存在的key将会被当做空字符串来处理。

### 返回值

返回字符串里面第一个被设置为1或者0的bit位。

如果我们在空字符串或0字节的字符串里面查找bit为1的内容，那么结果将返回-1。

如果我们在字符串里面查找bit为0而且字符串只包含1的值时，将返回字符串最右边的第一个空位。如果有一个字符串是三个字节的值为0xff的字符串，那么命令`bitpos key 0`将返回24，因为0-23位都是1。

基本上，我们可以把字符串看成右边有无数个0。

然而，如果你用指定startt和end范围进行查找指定值时，如果该范围内没有对应值，结果将返回-1。

### Command Operation

```shell
127.0.0.1:6379> setbit w 4 1
(integer) 0
127.0.0.1:6379> setbit w 8 1
(integer) 0
127.0.0.1:6379> bitpos w 0 #查找字符串里第一个bit为0的位置
(integer) 0
127.0.0.1:6379> bitpos w 1 #查找字符串里第一个bit为1的位置
(integer) 4
127.0.0.1:6379> bitpos w 1 3 7 #查找字符串里第三个字节(24)至第7个字节(56)里第一个bit为1的位置
(integer) -1
127.0.0.1:6379> bitpos n 1 #查找一个不存在字符串里bit值为1的位置
(integer) -1
127.0.0.1:6379> bitpos w 1 1 #查找字符串里bit值，从第1个字节(8)开始的位置
(integer) 8
127.0.0.1:6379> bitpos w 1 0 #查找字符串里bit值，从第0个字节(0)开始的位置
(integer) 4
```

注意：指定的位置是按字节为基本单位，即8个bit位。

### Code Operation

```java
/**
 * 查询今年签到的第一天
 */
private void bitpos1(){
    Long execute = stringRedisTemplate.execute((RedisCallback<Long>) connection ->
                                               connection.bitPos(stringRedisTemplate.getStringSerializer().serialize("bitmap"), true)
                                              );
    assert Objects.nonNull(execute);
    System.out.printf("今年签到的第一天是第%d天。\n", ++execute);
}

/**
 * 查询第96天之后签到的第一天
 * 注意：range是以字节为基本单位
 */
private void botpos2(){
    Range<Long> range = Range.from(Range.Bound.inclusive(96/8L)).to(Range.Bound.inclusive(365/8L));
    Long execute = stringRedisTemplate.execute((RedisCallback<Long>) connection ->
                                               connection.bitPos(stringRedisTemplate.getStringSerializer().serialize("bitmap"), true, range)
                                              );
    System.out.printf("96天之后签到的第一天是第%d天。\n", execute);
}
```





## bitfiled

```shell
$ bitfield [GET type offset][SET type offset value][INCRBY type offset increment][OVERFLOW WRAP|SAT|FAIL]
```

> 起始版本：3.2.0
>
> 时间复杂度：O(1)

本命令会把redis字符串当做位数组，并能对变长位宽和任意未字节对齐的指定整型位域进行寻址。在实践中，可以使用该命令对一个有符号的5位整型数的1234位设置指定值，也可以对一个31位无符号整型数的4567位进行取值。类似地，在对指定的整数进行自增和自减操作，本命令可以提供有保证的，可配置的上溢和下溢处理操作。

`bitfield`命令能操作多字节位域，它会执行一系列操作，并返回一个响应数组，在参数列表中每个响应数组匹配响应的操作。

例如，下面的命令是对一个8位有符号整数偏移100位自增1，并获取4位服务号整数的值。

```shell
192.168.56.3:6380> bitfield bitmap incrby i5 100 1 get u4 0
1) (integer) -1
2) (integer) 10
```

提示：

1. 用`GET`指令对超出当前字符串长度的位（含key不存在的情况）进行寻址，执行操作的结果会对缺失部分的位（bits）赋值为0。
2. 用`SET`或`INCRBY`指令对超出当前字符串长度的位（含key不存在的情况）进行寻址，将会扩展字符串并对扩展部分进行补0，扩展方式包括：按需扩展、按最小长度扩展和按最大寻址能力扩展。

**支持子命令和整型**

下面是已支持的命令列表：

- `GET <type> <offset>`—返回指定位域。
- `SET <type> <offset> <value>`—设置指定位域的值并返回它的原值。
- `INCRBY <type> <offset> <increment>`—自增或自减（如果increment为负数）指定位域的值并返回它的新值。

还有一个命令通过设置溢出行为来改变调用`INCRBY`指令的后序操作：

- `OVERFLOW [wrap|sat|fail]`

  当需要一个整型时，有符号整型需在位数前加`i`，无符号在位数前加`u`。例如`u8`是一个8位的无符号整型，`i16`是一个16位的有符号整型。

  有符号整型最大支持64位。而无符号整型最大支持63位。对无符号整型的限制，是由于当前Redis协议不能在响应消息中返回64位无符号整数。

**位和位偏移**

`bitfield`命令有两种方式来指定位偏移。如果未定待数字的前缀，将会以字符串的第0位作为起始位。

不过，如果偏移量带有`#`前缀，那么指定的偏移量需要乘以整型的宽度，例如：

```shell
$ BITFIELD bitmap SET i8 #0 100 i8 #1 200
```

将会在第1个`i8`整数的偏移0位和第二个证书的偏移8位进行设值。如果想得到一个给定长度的普通整型数组，则不一定要在客户端进行计算。

**移除控制**

使用`OVERFLOW`命令，用户可以通过指定下列其中一种行为来调整自增或自减操作溢出（或下溢）后的行为：

- `WRAP`：回环算法，适用于有符号和无符号整型两种类型。对于无符号整型，回环计数将对整型最大值进行取模操作（C语言的标准行为）。对于有符号整型，上溢从最负的负数开始取数，下溢则从最大的证书开始取数，例如如果`i8`整型的值设为127，自加1后的值变为-128。
- `SAT`：饱和算法，下溢之后设置最小的整形值，上溢之后设为最大的整数值。例如，`i8`整型的值从120开始加10后，结果是127，继续增加，结果还是保持为127。下溢也是同理，但量结果值将会保持在最负的负数值。
- `FAIL`：失败算法，这种模式下，在检测到上溢或下溢时，不做任何操作。相应的返回值会设为NULL，并返回给调用者。

注意每种溢出（`OVERFLOW`）控制方法，仅影响紧跟在`INCRBY`命令后的指子命令，直到重新执行溢出（`OVERFLOW`）控制方法。

如果没有指定溢出控制方法，默认情况下，将使用`WRAP`算法。

```shell
127.0.0.1:6379> bitfield mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1 
1) (integer) 1
2) (integer) 1
127.0.0.1:6379> bitfield mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1 
1) (integer) 2
2) (integer) 2
127.0.0.1:6379> bitfield mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1 
1) (integer) 3
2) (integer) 3
127.0.0.1:6379> bitfield mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1 
1) (integer) 0
2) (integer) 3
```

### 返回值

本命令返回一个针对子命令给定位置的处理结果组成的数组。`OVERFLOW`子命令在相应消息中，不会统计结果的条数。

**动机**

本命令的动机是为了能够在单个大位图（large bitmap）中高效地存储多个小整数（或对键分成多个key，避免出现超大键），同时开放Redis提供的新使用案例，尤其是在实时分析领域。这种使用案例可以通过指定的溢出控制方法来支持。

**性能考虑**

通常，`BITFIELD`是一个非常快的命令，但是注意，对短字符串的远地址（fat bits）寻址，将会比在存在的位执行命令更加耗时。

**字节序**

`BITFIELD`命令使用的位图表现形式，可看作是从0位开始的，例如：把一个5位的无符号整数23，对一个所有位事先置0的位图，从第7位开始复制，其结果如下所示：

```
+--------+--------+
|00000001|01110000|
+--------+--------+
```

当偏移量和整型大小是字节边界对齐时，此时与大端模式（big endian）相同，但是，当字节边界为对齐时，那么理解字节序将变得非常重要。

### Code Operation

```java
private void bitfield(){
    //从第三位开始（从0开始，包含第三位），取8为无符号数
    BitFieldSubCommands.BitFieldType uint8 = BitFieldSubCommands.BitFieldType.UINT_8;
    BitFieldSubCommands.Offset offset = BitFieldSubCommands.Offset.offset(3);
    //从第二十位开始（从0开始，包含第20位），将接下来两位用无符号数5替换
    BitFieldSubCommands.BitFieldType uint2 = BitFieldSubCommands.BitFieldType.unsigned(2);
    BitFieldSubCommands.Offset offset20 = BitFieldSubCommands.Offset.offset(20);
    //从第2位开始（从0开始，包含第2位），对接下来的4为无符号数+1，溢出折返-warp
    BitFieldSubCommands.BitFieldType uint4 = BitFieldSubCommands.BitFieldType.signed(4);
    BitFieldSubCommands.Offset offset2 = BitFieldSubCommands.Offset.offset(2);
    BitFieldSubCommands.BitFieldIncrBy.Overflow wrap = BitFieldSubCommands.BitFieldIncrBy.Overflow.WRAP;
    BitFieldSubCommands.BitFieldIncrBy.Overflow sat = BitFieldSubCommands.BitFieldIncrBy.Overflow.SAT;
    BitFieldSubCommands.BitFieldIncrBy.Overflow fail = BitFieldSubCommands.BitFieldIncrBy.Overflow.FAIL;

    BitFieldSubCommands bitFieldSubCommands = BitFieldSubCommands.create()
        .get(uint8).valueAt(offset)
        .set(uint2).valueAt(offset20).to(5)
        .get(uint4).valueAt(offset2)
        .incr(uint4).valueAt(offset2).overflow(wrap).by(1)
        .incr(uint4).valueAt(offset2).overflow(sat).by(1)
        .incr(uint4).valueAt(offset2).overflow(fail).by(1)
        .incr(uint4).valueAt(offset2).overflow(fail).by(1)
        .incr(uint4).valueAt(offset2).overflow(fail).by(1)
        .incr(uint4).valueAt(offset2).overflow(fail).by(1)
        .incr(uint4).valueAt(offset2).overflow(fail).by(1)
        .incr(uint4).valueAt(offset2).overflow(fail).by(1)
        .incr(uint4).valueAt(offset2).overflow(fail).by(1)
        .incr(uint4).valueAt(offset2).overflow(fail).by(1)
        .incr(uint4).valueAt(offset2).overflow(wrap).by(1)
        .get(uint4).valueAt(offset2);

    System.err.println("bitfield bitmap get "+uint8.asString()+" "+offset.getValue());
    System.err.println("bitfield bitmap set "+uint2.asString()+" "+offset20.getValue()+" "+5);
    System.err.println("bitfield bitmap overflow wrap incrby "+uint4.asString()+" "+offset2.getValue()+" "+1);
    List<Long> longs = stringRedisTemplate.opsForValue().bitField(KEY, bitFieldSubCommands);
    longs.forEach(l->{
        System.out.println(l+" : "+Integer.toBinaryString(l.intValue()));
    });
}
```

输出结果

```
bitfield bitmap get u8 3
bitfield bitmap set u2 20 5
bitfield bitmap overflow wrap incrby u4 2 1
8 : 1000
0 : 0
0 : 0
1 : 1
2 : 10
3 : 11
4 : 100
5 : 101
6 : 110
7 : 111
8 : 1000
9 : 1001
10 : 1010
11 : 1011
11 : 1011
```

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

