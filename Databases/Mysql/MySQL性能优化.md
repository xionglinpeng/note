# MySQL性能优化

一、优化简介

MySQL数据库优化是多方面的，原则是减少系统的瓶颈，减少资源的占用，增加系统的反应速度。

例如：

- 通过优化文件系统，提高磁盘I\O的读写速度；
- 通过优化操作系统调度策略，提高MySQL在高负荷情况下的负载能力；
- 优化表结构，索引，查询语句等使查询响应更快。

查询MySQL性能参数

语法：

```mysql
show status like 'value';
```

一些常用的性能参数：

- `connections`：连接MySQL服务器的次数。
- `uptime`：MySQL服务器的上线时间。
- `slow_queries`：慢查询次数
- `com_select`：查询操作的次数。
- `com_insert`：插入操作的次数。
- `com_update`：更新操作的次数。
- `com_delete`：删除操作的次数。

## 二、优化查询

### 2.1. 分析查询语句

MySQL提高了`explain`和`describe`语句，用来分析查询语句。

语法：

```mysql
explain [extended] select select_option
describe select select_option
# describe可缩写为desc
desc select select_option
```

示例：

```mysql
mysql> explain select * from user;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | user  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    2 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
1 row in set, 1 warning (0.14 sec)
```

说明：

- `id`：select标识符，这是select的查询序列号。

- `select_type`：

  表示select语句的类型。它可以是一下几种取值：

  - SIMPLE表示简单查询，其中不包括连接查询和子查询；
  - PRIMARY表示主查询，或者是最外层查询语句；
  - UNION表示连接查询的第2个或后面的查询语句；
  - DEPENDENT UNION表示连接查询中的第2个或后面的SELECT语句，取决于外面的查询；
  - UNION RESULT表示连接查询的结果；
  - SUBQUERY表示子查询中的第1个SELECT语句；
  - DEPENDENT SUBQUERY表示子查询中的第1个SELECT，取决于外面的查询；
  - DERIVED表示导出表的SELECT（FROM子句的子查询）。

- `table`：表示查询的表

- `partitions`：

- `type`：表示表的连接类型。

  下面按照从最佳类型到最差类型的顺序给出各种连接类型：

  1. system
  2. const
  3. eq_ref
  4. ref
  5. ref_or_null
  6. index_merge
  7. unique_subquery
  8. index_subquery
  9. range
  10. index
  11. ALL

- `possible_keys`：指出MySQL能使用那个索引在该表中找到行。如果该列时NULL，则没有相关的索引。在这种情况下，可以通过检查WHERE子句看它是否引用某些列或适合索引的列来提高查询性能。如果是这样，可以创建合适的索引来提高查询的性能。

- `key`：表示查询实际使用到的索引，如果没有选择索引，该列的值是NULL。要想强制MySQL使用或忽视`possible_keys`列中的索引，在查询中使用`FORCE INDEX`、`USE INDEX`或者`IGNORE INDEX`。**参见SELECT语法**。

- `key_len`：表示MySQL选择的索引字段按字节计算的长度，如果键是NULL，则长度为NULL。注意通过`key_len`值可以确定MySQL将实际使用一个多列索引中的几个字段。

- `ref`：表示使用哪个列或常数与索引一起来查询记录。

- `rows`：显示MySQL在表中进行查询时必须检查的行数。

- `filtered`：

- `Extra`：表示MySQL在处理查询时的详细信息。

### 2.2. 索引对查询速度的影响

如果查询时没有使用索引，查询语句将扫描表中的所有记录。在数据量大的情况下，这样查询的速度会很慢，如果使用索引进行查询，查询语句可以根据索引快速定位到待查询的记录，从而减少查询的记录数，达到提高查询速度的目的。

**示例：**

创建表，并插入一些数据

```mysql
CREATE TABLE `optimize` (
  `id` BIGINT UNSIGNED NOT NULL,
  `name` VARCHAR(50) DEFAULT NULL,
  `nickname` VARCHAR(50) DEFAULT NULL,
  `age` TINYINT DEFAULT NULL,
  PRIMARY KEY (`id`)
)

INSERT INTO `optimize` VALUES('1','周瑜','公瑾 ','21');
INSERT INTO `optimize` VALUES('2','刘备','玄德 ','22');
INSERT INTO `optimize` VALUES('3','许诸','虎痴','23');
INSERT INTO `optimize` VALUES('4','曹操','孟德','24');
INSERT INTO `optimize` VALUES('5','左慈','元放 ','25');
INSERT INTO `optimize` VALUES('6','吕布','奉先','26');
INSERT INTO `optimize` VALUES('7','貂蝉','任红昌','27');
INSERT INTO `optimize` VALUES('8','诸葛亮','孔明','28');
INSERT INTO `optimize` VALUES('9','赵云','子龙','29');
INSERT INTO `optimize` VALUES('10','关羽','云长','20');
```

在没有索引的情况下查询

```mysql
mysql> explain select * from `optimize` where `name`='周瑜' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: optimize
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 10.00
        Extra: Using where
```

可以看到，`type`为`ALL`，扫描的行数`rows`为10，即进行了全表扫描。

现在为name设置索引

```mysql
CREATE INDEX index_name ON `optimize`(`name`);
```

再次查看执行策略

```mysql
mysql> explain select * from `optimize` where `name`='周瑜' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: optimize
   partitions: NULL
         type: ref
possible_keys: index_name
          key: index_name
      key_len: 203
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

可以明显看到type变为ref了，使用了索引index_name，并且扫描的行数rows为1。相比而言，它的效率明显大大高于没有添加索引的时候。

### 2.3. 使用索引查询

索引可以提高查询的速度，但并不是使用带有索引的字段查询时索引都会起作用。

使用索引有几种特殊情况，在这些情况下有可能使用带有索引的字段查询时索引不会起作用。

> 示例以2.2节的表为基础操作。

#### 2.3.1、使用LIKE关键字的查询语句

在使用`LIKE`关键字进行查询的查询语句中，如果匹配字符串的第一个字符为“`%`”，索引不会起作用。所有“`%`”不在第一个位置，索引才会起作用。

示例：

匹配字符串的第一个字符为“`%`”：

```mysql
mysql> EXPLAIN SELECT * FROM `optimize` WHERE `name` LIKE '%周' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: optimize
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 11.11
        Extra: Using where
```

匹配字符串的第一个字符不为“`%`”：

```mysql
mysql> EXPLAIN SELECT * FROM `optimize` WHERE `name` LIKE '周%' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: optimize
   partitions: NULL
         type: range
possible_keys: index_name
          key: index_name
      key_len: 203
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition
```

#### 2.3.2、使用多列索引的查询语句

MySQL可以创建多列索引（一个索引可以包含16个字段）。对于多列索引，**只有查询条件中使用了这些字段中的第一个字段时索引才会起作用**。

示例：

首先创建多列索引

```mysql
CREATE INDEX index_nickname_age ON `optimize`(`nickname`,`age`);
```

使用多列索引的第一个字段查询（使用了索引）：

```mysql
mysql> EXPLAIN SELECT * FROM `optimize` WHERE `nickname` = '公瑾 ' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: optimize
   partitions: NULL
         type: ref
possible_keys: index_nickname_age
          key: index_nickname_age
      key_len: 203
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

使用多列索引的非第一个字段查询（没有使用索引）：

```mysql
mysql> EXPLAIN SELECT * FROM `optimize` WHERE `age` = '20' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: optimize
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 10.00
        Extra: Using where
```

#### 2.3.3、使用OR关键字的查询语句

查询语句的查询条件中只有OR关键字，且OR前后的两个条件中的列都是索引时，查询中才使用索引；否则，查询将不使用索引。

示例：

`age`字段没有索引，查询 - 执行策略没有使用索引。

```mysql
mysql> EXPLAIN SELECT * FROM `optimize` WHERE `name` = '周瑜' OR `age` = 20 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: optimize
   partitions: NULL
         type: ALL
possible_keys: index_name
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 19.00
        Extra: Using where
```

为`age`字段添加索引

```mysql
CREATE INDEX index_age ON `optimize`(`age`);
```

再次查询

```mysql
mysql> EXPLAIN SELECT * FROM `optimize` WHERE `name` = '周瑜' OR `age` = 20 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: optimize
   partitions: NULL
         type: index_merge
possible_keys: index_name,index_age
          key: index_name,index_age
      key_len: 203,2
          ref: NULL
         rows: 2
     filtered: 100.00
        Extra: Using union(index_name,index_age); Using where
```

### 2.4. 优化子查询

MySQL从4.1版本开始支持子查询，使用子查询可以进行SELECT语句的嵌套查询，即一个SELECT查询的结果作为另一个SELECTL语句的条件。子查询可以一次性完成很多逻辑上需要多个步骤才能完成的SQL操作。

子查询可以一次性完成很多逻辑上需要多个步骤才能完成的SQL操作。子查询虽然可以使查询语句很灵活，但执行效率不高。执行子查询时，MySQL需要为内层查询语句的查询结果建立一个临时表。然后外层查询语句从临时表中查询记录。查询完毕后，再撤销这些临时表。因此，子查询的速度会受到一定的影响。如果查询的数据量比较大，这种影响就会随之增大。

在MySQL中，可以使用连接（JOIN）查询来替代子查询。连接查询不需要建立临时表，其速度比子查询要快，如果查询汇总使用索引，性能会更好。连接之所以更有效率，是因为MySQL不需要在内存中创建临时表来完成查询工作。

## 三、优化数据库结构

### 3.1、将字段很多的表分解成多个表

对于字段较多的表，如果有些字段的使用频率很低，可以将这些字段分离出来，形成新表。因为当一个表的数据量很大时，会由于使用频率低的字段的存在而变慢。如果同时需要这些信息，可以通过联合查询。

### 3.2、增加中间表

对于需要经常联合查询的表，可以建立中间表，以提高查询效率。通过建立中间表，把需要经常联合查询的数据插入到中间表中，然后将原来的联合查询改为对中间表的查询，以此来提高查询效率。

### 3.3、增加冗余字段

设计数据库表时应尽量遵循范式理论规约，尽可能减少冗余字段，让数据库设计看起来精致、优雅。但是，合理的加入冗余字段可以提高查询速度。

> 提示
>
> 冗余字段会导致一些问题。比如，冗余字段的值在一个表中被修改了，就要想办法在其他表中更新该字段，否则就会使原本一致的数据变的不一致。分解表、增加中间表和增加冗余字段都浪费了一定的磁盘空间。从数据库性能的角度来看，为了提高查询速度而增加少量的冗余大部分时候是可以接受的。是否通过增加荣誉来提高数据库性能，就要根据实际需求综合分析。

### 3.4、优化插入记录的速度

插入记录时，影响插入速度的主要是索引、唯一性校验、一次插入记录数等。根据这些情况，可以分别进行优化。

#### 3.4.1、对于MyISAM引擎的表优化

常见的优化方法如下：

1. **禁用索引**

   对于非空表，插入记录时，MySQL会根据表的索引对插入的记录建立索引。如果插入大量数据，建立索引会降低插入记录的速度。为了解决这种情况，可以在插入记录之前禁用索引，数据插入完毕之后再开启索引。

   禁用索引：

   ```mysql
   ALTER TABLE TABLE_NAME DISABLE KEYS;
   ```

   启用索引：

   ```mysql
   ALTER TABLE TABLE_NAME ENABLE KEYS;
   ```

   对于空表批量导入数据，则不需要进行此操作，因为MyISAM引擎的表示在导入数据之后才建立索引的。

2. **禁用唯一性检查**

   插入数据时，MySQL会对插入的记录进行唯一性校验。这种唯一性校验也会降低插入记录的速度。为了降低这种情况对查询速度的影响，可以在插入记录之前禁用唯一性校验，等到记录插入完毕之后在开启。

   禁用唯一性校验：

   ```mysql
   SET unique_checks=0;
   ```

   启用唯一性校验：

   ```mysql
   SET unique_checks=1;
   ```

3. **使用批量插入**

   插入多条记录时，可以使用一条INSERT语句插入一条记录；也可以使用一条INSERT语句插入多条记录。

   一条INSERT语句插入一条记录情形：

   ```mysql
   INSERT INTO `user` VALUES ('','','','');
   INSERT INTO `user` VALUES ('','','','');
   INSERT INTO `user` VALUES ('','','','');
   ```

   一条INSERT语句插入多条记录情况：

   ```mysql
   INSERT INTO `user` VALUES ('','','',''),('','','',''),('','','','')
   ```

   第二种情形的插入速度比第一种情形快。

4. **使用LOAD DATA INFILE批量导入**

   当需要批量导入数据时，如果能用LOAD DATA INFILE语句，就尽量使用。因为LOAD DATA INFILE语句的导入速度比INSERT语句快。

#### 3.4.2、对于InnoDB引擎的表优化

常见的优化方法如下：

1. **禁用唯一性检查**

   跟MyISAM引擎的表优化法法一样。

2. **禁用外键检查**

   插入数据之前执行禁止对外键的检查，数据插入完成之后在恢复对外键的检查。

   禁用外键检查：

   ```mysql
   SET foreign_key_checks=0;
   ```

   启用外键检查：

   ```mysql
   SET foreign_key_checks=1;
   ```

3. **禁止自动提交**

   插入数据之前执行禁止事务的自动提交，数据插入完成之后在恢复自动提交操作。

   禁用事务自动提交：

      ```mysql
   SET autocommit=0;
      ```

   启用事务自动提交：

      ```mysql
   SET autocommit=1;
      ```

### 3.5、分析表、检查表和优化表

MySQL提供了分析表、检查表和优化表的语句。

- 分析表主要是分析关键字的分布；
- 检查表主要是检查表中是否存在错误；
- 优化表主要是消除删除或者更新造成的空间浪费。

#### 3.5.1、分析表

MySQL中提供了`ANALYZE TABLE`语句来分析表。

语法如下：

```mysql
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [,tbl_name]...
```

`LOCAL`关键字是`NO_WRITE_TO_BINLOG`关键字的别名，二者都是执行过程不写入二进制日志，`tbl_name`为分析表的表名，可以一个或多个。

使用ANALYZE TABLE分析表的过程中，数据库系统会自动对表加一个只读锁。在分析期间，只能读取表中的记录，不能更新和插入记录。ANALYZE TABLE语句能够分析InnoDB、BDB和MyISAM类型的表。

示例

```mysql
mysql> ANALYZE TABLE `user`;
+-------------------+---------+----------+----------+
| Table             | Op      | Msg_type | Msg_text |
+-------------------+---------+----------+----------+
| empty-window.user | analyze | status   | OK       |
+-------------------+---------+----------+----------+
```

- `Table`：表示分析的表的名称。
- `Op`：表示执行的操作。analyze表示进行分析操作。
- `Msg_type`：表示信息的类型，其值通常是状态（status）、信息（info）、注意（note）、警告（warning）和错误（error）之一。
- `Msg_text`：显示信息。

#### 3.5.2、检查表

MySQL中可以使用CHECK TABLE语句来检查表。CHECK TABLE语句能够检查InnoDB和MyISAM类型的表是否存在错误。对于MyISAM类型的表，CHECK TABLE语句还会更新关键字统计数据。而且，CHECK TABLE也可以检查视图是否有错误，比如在视图定义中被引用的表已经不存在。

语法如下：

```mysql
CHECK TABLE  tbl_name [,tbl_name]... [OPTION]...
	OPTION = {quick | fast | medium | extended | changed}
```

- `tbl_name`是表名
- `OPTION`参数有5个取值：
  1. `quick`：不扫描行，不检查错误的连接。
  2. `fast`：只检查没有被正确关闭的表。
  3. `medium`：扫描行，以验证被删除的连接是有效的。也可以计算各行的关键字检验和，并使用计算出的校验和验证这一点。
  4. `extended`：对每行的所有关键字进行一个全面的关键字查找。这可以确保表示100%一致的，但是花的时间较长。
  5. `changed`：只检查上次检查后被更改的表和没有被正确关闭的表。

`OPTION`只对MyISAM类型的表有效，对InnoDB类型的表无效。

CHECK TABLE语句在执行过程中也会给表加上只读锁。

示例

```mysql
mysql> CHECK TABLE `user`;
+-------------------+-------+----------+----------+
| Table             | Op    | Msg_type | Msg_text |
+-------------------+-------+----------+----------+
| empty-window.user | check | status   | OK       |
+-------------------+-------+----------+----------+
```

#### 3.5.3、优化表

MySQL中使用OPTIMIZE TABLE语句来优化表。该语句对InnoDB和MyISAM类型的表都有效。但是OPTIMIZE TABLE语句只能优化表中VARCHAR、BLOB和TEXT类型的字段。

语法如下：

```mysql
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [,tbl_name]...
```

通过`OPTIMIZE TABLE`语句可以消除删除和更新造成的文件碎片。`OPTIMIZE TABLE`语句在执行过程中也会给表加上只读锁。

示例：

```mysql
mysql> OPTIMIZE TABLE `user`;
+-------------------+----------+----------+-------------------------------------------------------------------+
| Table             | Op       | Msg_type | Msg_text                                                          |
+-------------------+----------+----------+-------------------------------------------------------------------+
| empty-window.user | optimize | note     | Table does not support optimize, doing recreate + analyze instead |
| empty-window.user | optimize | status   | OK                                                                |
+-------------------+----------+----------+-------------------------------------------------------------------+
```

## 四、优化MySQL服务器

优化MySQL服务器主要从两方面来优化：

1. 对硬件进行优化；
2. 对MySQL服务的参数进行优化；

### 4.1、优化服务器硬件

服务器的硬件性能直接决定着MySQL数据库的性能。优化策略有如下几种方式：

1. 配置较大的内存。足够大的内存是提高MySQL数据库性能的方法之一。内存的速度比磁盘I/O快得多，可以通过增加系统的缓冲区容量使数据在内存停留的时间更长，以减少磁盘I/O。
2. 配置高速磁盘系统，以减少读盘的等待时间，提高响应速度。
3. 合理分布磁盘I/O，把磁盘I/O分散在多个设备上，以减少资源竞争，提高并行操作能力。
4. 配置多处理器。MySQL是多线程的数据库，多处理器可同时执行多个线程。

### 4.2、优化MySQL的参数

MySQL服务的配置参数都在my.cnf或者my.ini文件的[MySQLd]组中。

1. `key_buffer_size`：表示索引缓冲区的大小。索引缓冲区所有的线程共享。增加索引缓冲区可以得到更好处理的索引（对所有读和多重写）。当然，这个值也不是越大越好，它的大小取决于内存的大小。如果这个值太大，导致操作系统频繁换页，也会降低系统性能。

2. `table_cache`：表示同时打开的表的个数。这个值越大。能够同时打开的表的个数越多。这个值不是越大越好，因为同时打开的表太多会影响操作系统的性能。

3. `query_cache_size`：表示查询缓冲区的大小。该参数需要和`query_cache_type`配合使用。

   - 当`query_cache_type=0`时，所有的查询都不使用查询缓冲区。

   - 当`query_cache_type=1`时。所有的查询都将使用查询缓冲区。除非在查询语句中指定`SQL_NO_CACHE`，如果`SELECT SQL_NO_CACHE * FROM tbl_name`。

   - 当`query_cache_type=2`时，只有在查询语句中使用`SQL_CACHE`关键字时，查询才会使用查询缓冲区。
   
     使用查询缓冲区可以提高查询的速度，这种方式只适用于修改操作少且经常执行相同的查询操作的情况。
   
4. `sort_buffer_size`：表示排序缓存区的大小。这个值越大，进行排序的速度越快。

5. `read_buffer_size`：表示每个线程连续扫描时为扫描的每个表分配的缓冲区的带大小（字节）。当线程从表中连续读取记录时需要用到这个缓冲区。`SET SESSION read_buffer_size=N`可以临时设置改参数的值。

   ```mysql
   mysql> show variables like 'read_buffer_size';
   +------------------+--------+
   | Variable_name    | Value  |
   +------------------+--------+
   | read_buffer_size | 131072 |
   +------------------+--------+
   ```

6. `read_rnd_buffer_size`：表示为每个线程保留的缓冲区的大小，与`read_buffer_sze`相似。但主要用于存储按特定顺序读取出来的记录。也可以用`SET SESSION read_rnd_buffer_size=n`来临时设置该参数的值。如果频繁进行多次连续扫描，可以增加该值。

7. `innodb_buffer_pool_size`：表示InnoDB类型的表和索引的最大缓存。这个值越大，查询的速度就会越快，但是这个值太大会影响操作系统的性能。

8. `max_connections`：表示数据库的最大连接数。这个连接数不是越大越好，因为这些连接会浪费内存的资源。过多的连接可能会导致MySQL服务器僵死。

9. `innodb_flush_log_at_trx_commit`：表示何时将缓冲区的数据写入日志文件，并且将日志文件写入磁盘中。该参数对于innoDB引擎非常重要。该参数有3个，分别是0、1和2：

   - 值为0时：表示每隔1秒将数据写入日志文件并将日志文件写入磁盘；

   - 值为1时：表示每次提交事务时将数据写入日志文件并将日志文件写入磁盘；

   - 值为2时：表示每次提交事务时将数据写入日志文件，每个1秒将日志文件写入磁盘。

     该参数的默认值为1。默认值1安全性最高，但是每次事务提交或事务外的指令都需要把日志写入（flush）硬盘，是比较费时的；0值更快一点，但是安全方便比较差；2值日志仍然会每秒写入到硬盘，所以即使出现故障，一般也不会丢失超过1~2秒的更新。

10. `back_log`：表示在MySQL暂时停止回答新请求之前的短时间内，多少个请求可以被存放在堆栈中。换句话说，该值表示对到来的TCP/IP连接的侦听队列的大小。只有期望在一个短时间内有很多连接时才需要增加该参数的值。操作系统在这个队列大小上也有限制，设定`back_log`高于操作系统的限制将是无效的。

11. `interactive_timeout`：表示服务器在关闭连接前等待行动的秒数。

12. `sort_buffer_size`：表示每个需要进行排序的线程分配的缓冲区的大小。增加这个参数的值可以提高`ORDER BY`或`GROUP BY`操作的速度，默认数值是2097144字节（约2MB）。

13. `thread_cache_size`：表示可以复用的线程的数量。如果有很多新的线程，为了提高性能可以增大该参数的值。

14. `wait_timeout`：表示服务器在关闭一个连接时等待行动的秒数，默认数值是28800。

15. `innodb_log_buffer_size`：

16. `innodb_log_file_size`：



































