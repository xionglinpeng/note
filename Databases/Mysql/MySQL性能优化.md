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





































