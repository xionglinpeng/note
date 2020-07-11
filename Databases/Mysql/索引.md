# 索引

## 1、索引简介

索引是对数据库表中一列或多列值的值进行排序的一种结构，使用索引可提高数据库中特定数据的查询速度。

### 1.1、索引的含义和特点

索引是一个单独的，存储在磁盘上的数据结构，包含着对数据表里所有记录的引用指针。使用索引可以快速找出在某个或多个列中有一特定值的行，所有MySQL列类型都可以被索引，对相关列使用索引是提高查询操作速度的最佳途径。

索引是在存储引擎中实现的，因此，每种存储引擎的索引都不一定完全相同，并且每种存储引擎也不一定支持所有索引类型。根据存储引擎定义每个表的最大索引数和最大索引长度。所有存储引擎支持每个表至少16个索引，总索引长度至少256字节。大多数存储引擎有更高的限制。MySQL中索引的存储类型有两种，即Btree和Hash，具体和表的存储引擎相关；MyISAM和InnoDB存储引擎支持Btree索引；Memory和Heap存储引擎可以支持Hash和Btree索引。

索引的优点：

1. 通过创建唯一索引，可以保证数据库表中每一行数据的唯一性。
2. 可以大大加快数据的查询速度，这也是创建索引的主要原因。
3. 在实现数据的参考完整性方面，可以加速表和表之间的连接。
4. 在使用分组和排序子句进行数据查询时，也可以显著减少查询中分组和排序的时间。

索引的缺点：

1. 创建索引和维护索引要耗费时间，并且随着数据量的增加所耗费的时间也会增加。
2. 索引需要占磁盘空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果有大量的索引，索引文件可能比数据文件更快达到最大文件尺寸。
3. 当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度。

### 1.2、索引的分类

1. 普通索引和唯一索引

   普通索引是MySQL中的基本索引类型，允许在定义索引的列中插入重复值和空值。

   唯一索引要求索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。主键索引是一种特殊的唯一索引，不允许有空值。

1. 单列索引和组合索引

   单列索引即一个索引只包含单个列，一个表可以有多个单列索引。

   组合索引是指在多个字段组合上创建的索引，只有在查询条件中使用了这些字段的左边字段时，索引才会被使用。使用组合索引时遵循最左前缀集合。

2. 全文索引

   全文索引类型为FULLTEXT，在定义索引的列上支持值的全文查找，允许在这些索引列中插入重复值和空值。全文索引可以在CHAR、VARCHAR或者TEXT类型的列上创建。MySQL中只有MyISAM存储引擎支持全文索引。

3. 空间索引

   空间索引是对空间数据类型字段建立的索引，MySQL中的空间数据类型由4中，分别是geometry、point、linestring和polygon。MySQL使用spatial关键字进行扩展，使得能够创建正规索引类似的语句创建空间索引。创建空间索引的列，必须将其声明为NOT NULL，空间索引只能在存储引擎为MyISAM的表中创建。

### 1.3、索引的设计原则

索引设计不合理或者缺少索引都会对数据库和应用程序的性能造成障碍。高效的索引对于获得良好的性能非常重要。设计索引时，应该考虑一下准则：

1. 索引并非越多越好。一个表中如有大量的索引，不仅占用磁盘空间，还会影响INSERT、DELETE、UPDATE等语句的性能，因为在表中的数据更改时，索引也会进行调整和更新。
2. 避免对经常更新的表进行过多的索引，并且索引中的列要尽可能少。应该经常用于查询的字段创建索引，但要避免添加不必要的字段。
3. 数据量小的表最好不要使用索引。由于数据较少，查询花费的时间可能比遍历索引的时间还要短，索引可能不会产生优化效果。
4. 在条件表达式中经常用到的不同值较多的列上建立索引，在不同值很少的列上不要建立索引。例如在学生表的“性别”字段上只有“男”和“女”两个不同值，因此就无须建立索引，如果建立索引，不能不会提高查询效率，反而会严重降低数据更新速度。
5. 当唯一性是某种数据本身的特征时，指定唯一索引。使用唯一索引需要能确保定义的列的数据完整性，以提高查询速度。
6. 在频繁进行排序或分组（即进行group by或order by操作）的列上建立索引。如果带排序的列有多个，可以在这些列上建立组合索引。

## 2、创建索引

### 2.1、创建表的时候创建索引

语法：

```mysql
CREATE TABLE table_name (
    [col_name data_type],
    [col_name data_type],
    [col_name data_type],
    ...
    [UNIQUE|FULLTEXT|SPATIAL] [INDEX|KEY] [index_name](col_name[length]) [ASC|DESC]
)
```

语法说明：

- `[UNIQUE|FULLTEXT|SPATIAL]`为可选参数，分别表示唯一索引、全文索引和空间索引。
- `[INDEX|KEY]`为同义词，两者作用相同，用来指定创建索引。
- `[index_name]`为索引名称，为可选参数，如果不指定，默认col_name为做引名称。
- `col_name`为需要创建索引的字段列，该列必须从数据表中定义的多个列中选择。
- `[length]`为可选参数，表示索引的长度，只有字符串类型的字段才能指定索引长度。
- `[ASC|DESC]`指定升序或者降序的索引值存储。

#### 2.1.1、创建普通索引

最基本的索引类型，没有唯一性之类的限制，其作用只是加快对数据的访问速度。

示例：

```mysql
CREATE TABLE `table` (
   id INT NOT NULL,
   `name` CHAR(30) NOT NULL,
   INDEX key_id(id)
);
```

查看表结构

```mysql
mysql> SHOW CREATE TABLE `table` \G;
*************************** 1. row ***************************
       Table: table
Create Table: CREATE TABLE `table` (
  `id` int NOT NULL,
  `name` char(30) NOT NULL,
  KEY `key_id` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```

#### 2.1.2、创建唯一索引

创建唯一索引的主要原因是减少查询索引列操作的执行时间，尤其是对比庞大的数据表。

它与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。

如果是组合索引，则列值的组合必须唯一。

示例：

```mysql
CREATE TABLE `table` (
   id INT NOT NULL,
   `name` CHAR(30) NOT NULL,
   UNIQUE INDEX unique_key_id(id)
);
```

查看表结构

```mysql
mysql> SHOW CREATE TABLE `table` \G;
*************************** 1. row ***************************
       Table: table
Create Table: CREATE TABLE `table` (
  `id` int NOT NULL,
  `name` char(30) NOT NULL,
  UNIQUE KEY `unique_key_id` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```

#### 2.1.3、创建单列索引

单列索引是在数据表中的某一个字段上创建的索引，一个表中可以创建多个单列索引。

前面的两个例子中创建的索引都为单列索引。

示例：

```mysql
CREATE TABLE `table` (
   id INT NOT NULL,
   `name` CHAR(30) NOT NULL,
   INDEX SingleIdx(id)
);
```

查看表结构

```mysql
mysql> SHOW CREATE TABLE `table` \G;
*************************** 1. row ***************************
       Table: table
Create Table: CREATE TABLE `table` (
  `id` int NOT NULL,
  `name` char(30) NOT NULL,
  KEY `SingleIdx` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```

#### 2.1.4、创建组合索引

组合索引是在多个字段上创建一个索引。

示例：

```mysql
CREATE TABLE `table` (
   id INT NOT NULL,
   `name` CHAR(30) NOT NULL,
   age INT NOT NULL,
   info VARCHAR(255) NOT NULL,
   INDEX MultiIdx(id,`name`,age)
);
```

查看表结构

```mysql
mysql> SHOW CREATE TABLE `table` \G;
*************************** 1. row ***************************
       Table: table
Create Table: CREATE TABLE `table` (
  `id` int NOT NULL,
  `name` char(30) NOT NULL,
  `age` int NOT NULL,
  `info` varchar(255) NOT NULL,
  KEY `MultiIdx` (`id`,`name`,`age`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```

组合索引可起几个索引的作用，但是使用时并不是随便查询那个字段都可以使用索引的，而是尊从“最左前缀”：利用索引中最左边的列集来匹配行，这样的列集称为最左前缀。例如，这里由id、name和age 3个字段构成的索引，索引行中安id、name、age的顺顺存放，索引可以搜索（id），(id,name)、（id,name,age）组合。如果列不构成索引最左面的前缀（如(age)或者(name,age)），那么MySQL不能使用局部索引，组合则不能使用索引查询。

使用EXPLAIN语句查询：

1. (id,name)组合

   ```mysql
   mysql> EXPLAIN SELECT * FROM `table` WHERE id=1 AND `name`='joe' \G;
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: table
      partitions: NULL
            type: ref
   possible_keys: MultiIdx
             key: MultiIdx
         key_len: 34
             ref: const,const
            rows: 1
        filtered: 100.00
           Extra: NULL
   1 row in set, 1 warning (0.00 sec)
   ```

   可以看到，使用了组合索引MultiIdx。

2. (name,age)组合

   ```mysql
   mysql> EXPLAIN SELECT * FROM `table` WHERE `name`='joe' AND age=2 \G;
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: table
      partitions: NULL
            type: ALL
   possible_keys: NULL
             key: NULL
         key_len: NULL
             ref: NULL
            rows: 1
        filtered: 100.00
           Extra: Using where
   1 row in set, 1 warning (0.00 sec)
   ```

   可以看到，没有使用索引。

#### 2.1.5、创建全文索引？？？

FULLTEXT全文索引可以用于全文搜索。只有

全文索引非常适合大型数据集，对于小的数据集，它的用处比较小。

示例：

```mysql
CREATE TABLE `table` (
   id INT NOT NULL,
   `name` CHAR(30) NOT NULL,
   age INT NOT NULL,
   info VARCHAR(255) NOT NULL,
   FULLTEXT FullTextIdx(info)
);
```

查看表结构

```mysql
mysql> SHOW CREATE TABLE `table` \G;
*************************** 1. row ***************************
       Table: table
Create Table: CREATE TABLE `table` (
  `id` int NOT NULL,
  `name` char(30) NOT NULL,
  `age` int NOT NULL,
  `info` varchar(255) NOT NULL,
  FULLTEXT KEY `FullTextIdx` (`info`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```



```mysql
mysql> EXPLAIN SELECT * FROM `table` WHERE info='joe' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: table
   partitions: NULL
         type: ALL
possible_keys: FullTextIdx
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

查看表结构



#### 2.1.6、创建空间索引？？

空间索引必须在MyISAM类型的表中创建，且空间类型的字段必须为非空。

注意：创建时指定空间类型字段为NOT NULL，并且表的存储引擎为MyISAM。

示例：

```mysql
CREATE TABLE `table` (
   id INT NOT NULL,
   g GEOMETRY NOT NULL,
   SPATIAL INDEX SpatialIdx(g)
) ENGINE=MYISAM;
```

查看表结构



### 2.2、在已经存在的表上创建索引

#### 2.2.1、使用ALTER TABLE语句创建索引

ALTER TABLE语法：

```mysql
ALTER TABLE tb_name ADD 
	[UNIQUE|FULLTEXT|SPATIAL] [INDEX|KEY] 
	[index_name](col_name[length]...) [ASC|DESC]
```

示例：

1. 创建表table

   ```mysql
   CREATE TABLE `table` (
      id INT NOT NULL,
      `name` CHAR(30) NOT NULL,
      phone VARCHAR(13) NOT NULL,
      `text` VARCHAR(255) NOT NULL,
      g GEOMETRY NOT NULL
   );
   ```
   
2. 添加索引

   ```mysql
   -- 创建普通|单列索引
   ALTER TABLE `table` ADD INDEX IndexIdx(id);
   -- 创建唯一索引
   ALTER TABLE `table` ADD UNIQUE INDEX UniqueIdx(`name`(30));
   -- 创建组合索引
   ALTER TABLE `table` ADD INDEX MiltiIdx(`name`(30),phone(13));
   -- 创建全文索引
   ALTER TABLE `table` ADD FULLTEXT INDEX FullTextIdx(`text`);
   -- 创建空间索引
   ALTER TABLE `table` ADD SPATIAL INDEX SpatialIdx(g);
   ```

3. 查看索引

   ```mysql
   SHOW INDEX FROM `table`;
   ```
   |Table|Non_unique|Key_name|Seq_in_index|Column_name|Collation|Cardinality|Sub_part|Packed|Null|Index_type|Comment|Index_comment|Visible|Expression|
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|table|0|UniqueIdx|1|name|A|0|NULL|NULL||BTREE|||YES|NULL|
|table|1|IndexIdx|1|id|A|0|NULL|NULL||BTREE|||YES|NULL|
|table|1|MiltiIdx|1|name|A|0|NULL|NULL||BTREE|||YES|NULL|
|table|1|MiltiIdx|2|phone|A|0|NULL|NULL||BTREE|||YES|NULL|
|table|1|SpatialIdx|1|g|A|0|32|NULL||SPATIAL|||YES|NULL|
|table|1|FullTextIdx|1|text|NULL|0|NULL|NULL||FULLTEXT|||YES|NULL|



`SHOW INDEX FROM <tbname>`字段说明：

1. `Table`：表示创建索引的表。
2. `Non_unique`：表示索引非唯一，1代表非唯一索引，0代表唯一索引。
3. `Key_name`：表示索引的名称。
4. `Seq_in_index`：表示该字段在索引中的位置，单列索引该值为1，组合索引为每个字段在索引定义中的顺序。
5. `Column_name`：表示定义索引的列字段。
6. `Collation`：
7. `Cardinality`：
8. `Sub_part`：表示索引的长度。
9. `Packed`：
10. `Null`：表示该字段是否能为空值。
11. `Index_type`：表示索引类型。
12. `Comment`：
13. `Index_comment`：
14. `Visible`：
15. `Expression`：


#### 2.2.2、使用CREATE INDEX创建索引

CREATE INDEX语法：

```mysql
CREATE [UNIQUE|FULLTEXT|SPATIAL] [INDEX|KEY] [index_name] ON table_name(col_name[LENGTH]...) [ASC|DESC]
```

示例：

创建表

```mysql
DROP TABLE IF EXISTS `table`;
CREATE TABLE `table` (
   id INT NOT NULL,
   `name` CHAR(30) NOT NULL,
   phone VARCHAR(13) NOT NULL,
   `text` VARCHAR(255) NOT NULL,
   g GEOMETRY NOT NULL
);
```

创建索引

```mysql
-- 创建普通|单列索引
CREATE INDEX IndexIdx ON `table`(id);
-- 创建唯一索引
CREATE UNIQUE INDEX UniqueIdx ON `table`(`name`(30));
-- 创建组合索引
CREATE INDEX MiltiIdx ON `table`(`name`(30),phone(13));
-- 创建全文索引
CREATE FULLTEXT INDEX FullTextIdx ON `table`(`text`);
-- 创建空间索引
CREATE SPATIAL INDEX SpatialIdx ON `table`(g);
```

### 2.3、Explain

使用EXPLAIN语句可以查看索引是否正在使用。

示例：

```mysql
EXPLAIN SELECT * FROM USER WHERE NAME='user2';
```

| id   | select_type | table | partitions | type | possible_keys | key  | key_len | ref   | rows | filtered | Extra |
| ---- | ----------- | ----- | ---------- | ---- | ------------- | ---- | ------- | ----- | ---- | -------- | ----- |
| 1    | SIMPLE      | USER  | NULL       | ref  | name          | name | 23      | const | 3    | 100.00   | NULL  |

字段说明：

1. `select_type`行指定所使用的select查询类型，这里的值为`SIMPLE`，表示简单的SELECT，不使用UNION或子查询。其他可能的取值有`PRIMARY`、`UNION`、`SUBQUERY`等。
2. `table`行指定数据库读取的数据表的名字，它们按被读取的先后顺序排列。
3. partitions
4. `type`行指定了数据表与其他数据表之间的关联关系，可能的取值有`system`、`const`、`eq_ref`、`ref`、`range`、`index`和`all`。
5. `possible_keys`行给出了MySQL在搜索数据记录时可选用的各个索引。
6. `key`行是MySQL实际选用的索引。
7. `key_len`行给出索引按字节计算的长度，`key_len`数值越小，表示越快。
8. `ref`行给出了关联关系中另一个数据表里的数据列名。
9. `rows`行是MySQL在执行这个查询时预计会从这个数据表里读取的数据行的个数。
10. filtered
11. `Extra`行提供了与关联操作有关的信息。

## 3、删除索引

### 3.1、使用ALTER TABLE删除索引

ALTER TABLE语法：

```mysql
ALTER TABLE tb_name DROP INDEX index_name;
```

示例：

```mysql
ALTER TABLE `table` DROP INDEX UniqueIdx;
```

> 提示：
>
> 添加`AUTO_INCREMENT`约束字段的唯一索引不能被删除。

### 3.2、使用DROP INDEX语句删除索引

DROP INDEX语法：

```mysql
DROP INDEX index_name ON tb_name;
```

示例：

```mysql
DROP INDEX UniqueIdx ON `table`;
```

> 提示：
>
> 删除表中的列时，如果要删除的列为索引的组成部分，则该列也会从索引中删除。如果组成索引的所有列都被删除，则整个索引将被删除。



## 4、支持降序索引





## 5、统计直方图



