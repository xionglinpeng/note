# scan

[SCAN](http://www.redis.cn/commands/scan.html)命令用于增量迭代一个集合元素。

每次执行都只返回少量的元素，因此该命令可用于生产环境。

由于scan命令是增量式迭代，有可能在增量迭代过程中，集合元素被修改，因此对于返回值无法完全准确的保证。

**基本原理**

通过游标（cursor）进行迭代，每次调用都需要使用上一次调用所返回的游标作为当前调用的游标参数。

当`SCAN命令的游标参数被设置为0时，表示服务器开始一次新的迭代。当SCAN命令返回的游标数为0时，表示迭代已结束。

可以通过参数`MATCH`设置迭代时需要匹配的KEY的模式。

默认情况下，每一次迭代最多可以迭代10个KEY（不论是否匹配），可以通过`COUNT`参数设置每次迭代的KEY数量。

**语法**

```shell
$ scan cursor [MATCH pattern] [COUNT count]
```

**特性**

1. <h5 style="color:#be0606">设置游标为0表示开始一次新的迭代，返回游标0表示迭代结束。</h5>

2. <h5 style="color:#be0606">同KEYS一样，也提供了模式匹配功能，通过参数MATCH设置。</h5>

3. <h5 style="color:#be0606">COUNT参数并不表示迭代返回的结果数量，而是表示遍历字典槽位（slot）的数量，即KEY的数量。</h5>

4. <h5 style="color:#be0606">单次迭代返回的结果为空并不表示迭代结束，而是看返回的游标是否为0。</h5>

5. <h5 style="color:#be0606">返回的结果可能会有重复，需要客户端去重（重要）。</h5>

6. <h5 style="color:#be0606">虽然SCAN命令迭代操作的复杂度是O(n)，但它是通过游标分步进行的，不会阻塞线程。</h5>

7. <h5 style="color:#be0606">SCAN命令的迭代是增量循环的，每次只会返回一小部分元素，所以不会使Redis假死。</h5>

8. <h5 style="color:#be0606">服务器不会保存游标状态，游标的唯一状态就是返回到客户端的游标结果，因此可以并发迭代。</h5>

**选项**

- `cursor`：设置迭代时的游标，0表示开始一次新的迭代。

  示例：

  ```shell
  127.0.0.1:6379> scan 0
  1) "11"
  2)  1) "g:1381446647003316225"
      2) "g:1381446646986539009"
      3) "g:1381446646919430146"
      4) "g:1381446646994927618"
      5) "s:g:1381446835214299137"
      6) "s:g:1381446835541454849"
      7) "g:1381446646927818753"
      8) "g:1381446646961373186"
      9) "g:1381446646936207362"
     10) "g:1381446646944595970"
  127.0.0.1:6379> scan 11
  1) "0"
  2) 1) "g:1381446646969761794"
     2) "g:1381446646583885826"
     3) "s:g:1381446835872804865"
  ```

- `MATCH`：设置迭代时需要匹配的KEY的模式。

  示例：

  ```shell
  127.0.0.1:6379> scan 0 match g*
  1) "11"
  2) 1) "g:1381446647003316225"
     2) "g:1381446646986539009"
     3) "g:1381446646919430146"
     4) "g:1381446646994927618"
     5) "g:1381446646927818753"
     6) "g:1381446646961373186"
     7) "g:1381446646936207362"
     8) "g:1381446646944595970"
  127.0.0.1:6379> scan 11 match g*
  1) "0"
  2) 1) "g:1381446646969761794"
     2) "g:1381446646583885826"
  ```

- `COUNT`：设置每次迭代的KEY数量。

  示例：

  ```shell
  127.0.0.1:6379> scan 0 match g* count 2
  1) "10"
  2) 1) "g:1381446647003316225"
     2) "g:1381446646986539009"
     
  127.0.0.1:6379> scan 10 match g* count 2
  1) "6"
  2) 1) "g:1381446646919430146"
     2) "g:1381446646994927618"
     
  127.0.0.1:6379> scan 6 match g* count 2
  1) "5"
  2) (empty list or set)
  
  127.0.0.1:6379> scan 5 match g* count 2
  1) "13"
  2) 1) "g:1381446646927818753"
     2) "g:1381446646961373186"
     
  127.0.0.1:6379> scan 13 match g* count 2
  1) "11"
  2) 1) "g:1381446646936207362"
     2) "g:1381446646944595970"
     
  127.0.0.1:6379> scan 11 match g* count 2
  1) "0"
  2) 1) "g:1381446646969761794"
     2) "g:1381446646583885826"
  ```

## 其他SCAN

`SCAN`是用于迭代一个仓库中所有KEY的命令，针对不同的数据类型，`Redis`还提供了其他`SCAN`命令，如下：

- `SCAN`命令用于迭代当前数据库中的KEY集合。
- `SSCAN`命令用于迭代`SET`集合中的元素。
- `HSCAN`命令用于迭代`HASH`类型中的键值对。
- `ZSCAN`命令用于迭代`ZSET`即可中的元素和元素对应的分值。

它们都是用于增量迭代一个集合中的元素。

##  返回值

`SCAN`、`SSCAN`、`HSCAN`、`ZSCAN`命令都返回两个元素的multi-bulk回复：

1. 第一个回复：字符串表示的无符号64位整数（游标）。
2. 第二个回复：包含本次迭代匹配到的元素：
   - `SCAN` — 每个元素都是一个`KEY`。
   - `SSCAN` — 每个元素都是一个`SET`集合的成员。
   - `HSCAN` — 每个元素都是一个键值对。
   - `ZSCAN` — 每个元素都是一个`ZSET`集合的元素——成员(member)+分值(score)）。



> **KEYS命令带来的问题**
>
> 在一个Redis数据库中，存储的KEY多达数十，甚至上百万千万，数据量是很多的。`KEYS`算法是遍历算法，复杂度是O(n)，也就是说，数据量越多，时间复杂度越高，因此当数据量很多时，执行`KEYS`命令将会花费大量的时间。因为Redis是单线程程序，顺序执行所有命令，所以当在执行KEYS命令时，其他命令必须等待`KEYS`执行完成之后才能执行，因此将会导致服务器阻塞甚至崩溃。

## Spring Data Redis `SCAN`实现

```java
@Slf4j
@Component
public class RedisHelper {

    private static final long DEFAULT_SCAN_COUNT = 1000L;

    @Autowired
    private RedisTemplate<Object, Object> redisTemplate;

    public void scan(Consumer<byte[]> consumer) {
        scan(null, 0L, consumer);
    }

    public void scan(String pattern, Consumer<byte[]> consumer) {
        scan(pattern, 0L, consumer);
    }

    public void scan(Long count, Consumer<byte[]> consumer) {
        scan(null, count, consumer);
    }

    public void scan(String pattern, long count, Consumer<byte[]> consumer) {
        redisTemplate.execute((RedisCallback<Object>) connection -> {
            ScanOptions.ScanOptionsBuilder scanOptionsBuilder = ScanOptions.scanOptions();
            if (StringUtils.hasText(pattern)) {
                scanOptionsBuilder.match(pattern);
            }
            scanOptionsBuilder.count(count > 0 ? count : DEFAULT_SCAN_COUNT);
            ScanOptions scanOptions = scanOptionsBuilder.build();
            log.debug("Scan parameter : {}", scanOptions.toOptionString());
            try (Cursor<byte[]> scan = connection.scan(scanOptions)) {
                log.trace("Scan CursorId = {} and Position = {}", scan.getCursorId(), scan.getPosition());
                scan.forEachRemaining(consumer);
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        });
    }

    public long count(String pattern) {
        ScanCount sc = new ScanCount();
        this.scan(pattern, b -> sc.incr());
        return sc.getCount();
    }

    public List<String> keys(String pattern) {
        List<String> keys = new LinkedList<>();
        this.scan(pattern, b -> keys.add(new String(b)));
        return keys;
    }


    @Data
    private static class ScanCount {

        int count;

        public void incr() {
            count++;
        }
    }

}
```



## `SSCAN`、`HSCAN`、`ZSCAN`

- `HASH`

  ```JAVA
  redisTemplate.opsForHash().scan(H key, ScanOptions options);
  ```

- `SET`

  ```JAVA
  redisTemplate.opsForSet().scan(H key, ScanOptions options);
  ```

- `ZSET`

  ```JAVA
  redisTemplate.opsForZSet().scan(H key, ScanOptions options);
  ```

  

