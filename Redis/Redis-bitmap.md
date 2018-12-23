# bitmap



## setbit







```java
/**
  * 模拟一年365天用户签到记录，1表示签到，0表示未签到。
  */
private void setbit(){
    ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();
    Random random = new Random();
    for (int i = 0; i < 365; i++) {
        int b = random.nextInt(0B10);
        operations.setBit("bitmap",i, b == 1);
        System.out.print(b);
    }
    System.out.println();
}
```

输出

```
01010100001100101000010100100111110101001111110110000110001000101111110000110100100101010001111010101001001101111100111110011111101110011101000011101001100010101000111011100010011100001110010010110001011010000100011110100111000101000100101001111101011000111110000001001100000000001000100001100110100100011100001010010100111110100000011101111111011100011100111000001
```





## getbit



## bitcount



## bitpos





## bitfiled





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

下面是使用`Spring-Data-Redis`API编写的代码实例：

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

