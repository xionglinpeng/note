# MySQL锁

## 1、MySQL锁概述

MySQL的不同存储引擎支持不同的锁机制：

- MyISAM和MEMORY支持表级锁。
- DBD支持页面锁和表级锁。
- InnoDB支持表级锁和行级锁。

三种锁的特性：

- 表级锁：开销小，加锁块；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。
- 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
- 页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

## 2、MyISAM表锁

准备一张表

```mysql
CREATE TABLE `user`(
 id INT UNSIGNED AUTO_INCREMENT,
 username VARCHAR(32) NOT NULL COMMENT '用户名',
 nickname VARCHAR(64) NOT NULL COMMENT '昵称',
 PRIMARY KEY(id)
) ENGINE=MYISAM
```

准备一些数据

```mysql
INSERT INTO `user` (username,nickname) VALUES  ('a123456','Jackson'),('b296007','Diana'),('d785415','Tom');
```

### 2.1、查询表级锁争用情况

使用`Table_locks_waited`和`Table_locks_immediate`状态变量来分析系统上的表锁定争夺情况：

```mysql
mysql> show status like 'table%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Table_locks_immediate      | 16    |
| Table_locks_waited         | 0     |
| Table_open_cache_hits      | 0     |
| Table_open_cache_misses    | 0     |
| Table_open_cache_overflows | 0     |
+----------------------------+-------+
5 rows in set (0.00 sec)
```

如果`Table_locks_waited`的值比较高，则说明存在着较严重的表级锁争用情况。

### 2.2、MySQL表级锁的锁模式

MySQL的表级锁模式有两种：表共享锁（Table Read Lock）和表独占锁（Table Write Lock）。

*锁模式的兼容性如下表：*

| 锁模式 | None | 读锁 | 写锁 |
| ------ | ---- | ---- | ---- |
| 读锁   | √    | √    | ×    |
| 写锁   | √    | ×    | ×    |

只有读读之间不会阻塞；其他无论是读写，还是写写都会阻塞，即它们之间是串行的。

*MyISAM写阻塞读示例：*

- 第一步：获得user表的WRITE锁定。

  session 1

  ```mysql
  mysql> lock table user write;
  Query OK, 0 rows affected (0.00 sec)
  ```

- 第二步：session 2对被锁定的user表进行查询被阻塞，需要等待锁被释放。

  session 2

  ```mysql
  mysql> select * from user;
  等待
  ```

- 第三步：session 1执行插入，更新，删除

  session 1

  ```mysql
  mysql> insert into user value (10,'K123456','Xavier');
  Query OK, 1 row affected (0.06 sec)
  
  mysql> update user set username='T123456' where id=10;
  Query OK, 1 row affected (0.06 sec)
  Rows matched: 1  Changed: 1  Warnings: 0
  
  mysql> delete from user where id=10;
  Query OK, 1 row affected (0.07 sec)
  ```

- 第四步：session 1释放表锁，session 2获得了锁，执行查询。

  session 1

  ```mysql
  mysql> unlock tables;
  Query OK, 0 rows affected (0.00 sec)
  ```

  session 2

  ```mysql
  mysql> select * from user;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Jackson  |
  |  2 | b296007  | Diana    |
  |  3 | d785415  | Tom      |
  +----+----------+----------+
  3 rows in set (1 min 9.98 sec)
  ```

### 2.3、如何加表锁

MyISAM在执行查询语句（`SELECT`）前，会自动给涉及的所有表加读锁。

在执行更新操作（`UPDATE`、`DELETE`、`INSERT`）前，会自动给涉及的表加写锁。

自动加锁的过程不需要用户干预，因此，用户一般不需要直接用`LOCK TABLE`命令给MyISAM表显式加锁。

*MyISAM读阻塞读示例：*

- 第一步：session 1获取读锁。

  session 1

  ```mysql
  mysql> lock table user read;
  Query OK, 0 rows affected (0.00 sec)
  ```

- 第二步：session 1和session 2都可以查询该表的记录。

  session 1 and session 2

  ```mysql
  mysql> select * from user;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Xavier   |
  |  2 | b296007  | Diana    |
  |  3 | d785415  | Tom      |
  +----+----------+----------+
  4 rows in set (0.00 sec)
  ```

- 第三步：session 1不能查询其他表的记录，但是session 2可以查询或者更新未锁定的表。

  session 1

  ```mysql
  mysql> select * from book;
  ERROR 1100 (HY000): Table 'book' was not locked with LOCK TABLES
  ```

  session 2

  ```mysql
  mysql> select * from book;
  +----+-------+
  | id | name  |
  +----+-------+
  |  1 | gengu |
  +----+-------+
  1 row in set (0.00 sec)
  
  mysql> update book set name = 'xlp';
  Query OK, 0 rows affected (0.00 sec)
  Rows matched: 1  Changed: 0  Warnings: 0
  ```

- 第四步：session 1如果对当前表执行INSERT、UPDATE和DELETE操作，都会提示错误。而session 2更新会阻塞等待。

  session 1

  ```mysql
  mysql> insert into user values (10,'K123456','Xavier');
  ERROR 1099 (HY000): Table 'user' was locked with a READ lock and can't be updated
  mysql> update user set nickname='Xavier' where id=1;
  ERROR 1099 (HY000): Table 'user' was locked with a READ lock and can't be updated
  mysql> delete from user where id=1;
  ERROR 1099 (HY000): Table 'user' was locked with a READ lock and can't be updated
  ```

  session 2

  ```mysql
  mysql> update user set nickname='Xavier' where id=1;
  等待
  ```

- 第五步：session 1是否锁；session 2获取到锁更新完成。

  session 1

  ```mysql
  mysql> unlock tables;
  Query OK, 0 rows affected (0.00 sec)
  ```

  session 2

  ```mysql
  mysql> update user set nickname='Xavier' where id=1;
  Query OK, 0 rows affected (1 min 2.18 sec)
  Rows matched: 1  Changed: 0  Warnings: 0
  ```

------

同时给多个表加锁，使用`LOCK TABLES`语法。在用`LOCK TABLES`给表显式加表锁时，必须同时取得所有涉及到表的锁，并且MySQL不支持锁升级。即包括如下两个特性：

1. 在执行`LOCK TABLES`后，只能访问显式加锁的这些表，不能访问未加锁的表；
2. 如果加的是读锁，那么只能执行查询操作，而不能执行更新操作。

其实，在自动加锁的情况下也基本如此，MyISAM总是一次获得SQL语句所需的全部锁。这正是MyISAM表不会出现死锁（Deadlock Free）的原因。

当使用`LOCK TABLES`时，不仅需要一次锁定用到的所有表，而且，同一个表在SQL语句中出现多少次，就要通过与SQL语句相同的别名锁定多少次，否则也会出错。

*同一个表在SQL语句中出现多少次示例：*

- 对user表加读锁。

  ```mysql
  mysql> lock table user read;
  Query OK, 0 rows affected (0.00 sec)
  ```

- 但是通过别名访问会提示错误。

  ```mysql
  mysql> select * from user a,user b where a.nickname=b.nickname;
  ERROR 1100 (HY000): Table 'a' was not locked with LOCK TABLES
  ```

- 需要对别名分别锁定。

  ```mysql
  mysql> lock table user as a read,user as b read;
  Query OK, 0 rows affected (0.00 sec)
  ```

- 按照别名查询可以正确执行。

  ```mysql
  mysql> select * from user a,user b where a.nickname=b.nickname;
  +----+----------+----------+----+----------+----------+
  | id | username | nickname | id | username | nickname |
  +----+----------+----------+----+----------+----------+
  |  1 | a123456  | Xavier   |  1 | a123456  | Xavier   |
  | 10 | K123456  | Xavier   |  1 | a123456  | Xavier   |
  |  2 | b296007  | Diana    |  2 | b296007  | Diana    |
  |  3 | d785415  | Tom      |  3 | d785415  | Tom      |
  |  1 | a123456  | Xavier   | 10 | K123456  | Xavier   |
  | 10 | K123456  | Xavier   | 10 | K123456  | Xavier   |
  +----+----------+----------+----+----------+----------+
  6 rows in set (0.05 sec)
  ```

### 2.4、并发插入

相关资料：https://dev.mysql.com/doc/refman/8.0/en/concurrent-inserts.html

总体而言，MyISAM表的读和写是串行的。但在一定条件下，MyISAM表也支持查询和插入操作的并发进行。

MyISAM存储引擎有一个系统变量`concurrent_insert`，专门用以控制其并发插入的行为，其值分别可以为

`AUTO`（或`1`），`NEVER`（或`0`）、`ALWAYS`（或`2`）。

> Note：
>
> 并发插入：指MyISAM允许在一个进程读表的同时，另一个进行从表尾插入记录。
>
> 空洞：MyISAM表中存在被删除的行。

- 如果`concurrent_insert`设置为`NEVER`（或`0`），则禁用并发插入。
- 如果`concurrent_insert`设置为`AUTO`（或`1`），当MyISAM表中没有空洞时，允许并发插入。
- 如果`concurrent_insert`设置为`ALWAYS`（或`2`），无论MyISAM表中有没有空洞，都允许并发插入。

`concurrent_insert`默认值为`AUTO（或1）`。

*查询concurrent_insert：*

```mysql
mysql> show variables like 'concurrent%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| concurrent_insert | AUTO  |
+-------------------+-------+
1 row in set, 1 warning (0.00 sec)
```

或者

```mysql
mysql> select @@concurrent_insert;
+---------------------+
| @@concurrent_insert |
+---------------------+
| AUTO                |
+---------------------+
1 row in set (0.00 sec)
```

------

并发插入示例（这里假设不存在空洞）：

- 第一步：session 1获的user表的`READ LOCAL`锁定

  session 1

  ```mysql
  mysql> lock table user read local;
  Query OK, 0 rows affected (0.03 sec)
  ```

- 第二步：session 1对user表执行INSERT、UPDATE和DELETE操作——不能执行。

  session 1

  ```mysql
  mysql> insert into user values (10,'K123456','Xavier');
  ERROR 1099 (HY000): Table 'user' was locked with a READ lock and can't be updated
  mysql> update user set nickname='Xavier' where id=1;
  ERROR 1099 (HY000): Table 'user' was locked with a READ lock and can't be updated
  mysql> delete from user where id=1;
  ERROR 1099 (HY000): Table 'user' was locked with a READ lock and can't be updated
  ```

- 第三步：session 2对user表执行INSERT和UPDATE操作——可以进行INSERT操作，但UPDATE操作会等待。

  session 2

  ```mysql
  mysql> insert into user values(10,'K123456','Xavier');
  Query OK, 1 row affected (0.07 sec)
  
  mysql> update user set nickname='Xavier' where id=1;
  等待
  ```

- 第四步：session 1查询刚才session 2插入的数据——不能访问其他session插入的记录。

  session 1

  ```mysql
  mysql> select * from user where id=10;
  Empty set (0.00 sec)
  ```

- 第五步：session 1是否锁——session 2获得锁，更新操作完成

  session 1

  ```mysql
  mysql> unlock tables;
  Query OK, 0 rows affected (0.00 sec)
  ```

  session 2

  ```mysql
  mysql> update user set nickname='Xavier' where id=1;
  Query OK, 1 row affected (1 min 59.25 sec)
  Rows matched: 1  Changed: 1  Warnings: 0
  ```

- 第六步：session 1查询刚才session 2插入的数据——可以查到。

  session 1

  ```mysql
  mysql> select * from user where id=10;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  | 10 | K123456  | Xavier   |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

可以利用MyISAM存储引擎的并发插入特性，来解决引用中对同一表查询和插入的锁争用。

- 方式1：将`concurrent_insert`系统变量设为2，总是允许并发插入。
- 方式2：通过定期在系统空闲时段执行`OPTIMIZE TABLE`语句来整理空间碎片，收回因删除记录而产生的中间空洞。

### 2.5、MyISAM的锁调度

MyISAM的锁调度的特性：

- MyISAM存储引擎的读锁和写锁是互斥的，读写操作是串行的。

- 一个进程请求某个MyISAM表的读锁，同时另一个进程也请求同一表的写锁，是写进行先获得锁。

- 即使读请求先到锁等待队列，写请求后到，写锁也会插到读锁请求之前！这是因为MySQL认为写请求一般比读请求更重要。

上面这些原因也正是MyISAM表不太适合于有大量更新操作和查询操作应用的原因，因为大量的更新操作会造成查询操作很获得读锁，从而可能永远阻塞。

可以通过一些设置来调节MyISAM的调度行为：

- 通过指定启动参数`low-priority-updates`，使MyISAM引擎默认给予读请求以优先的权利。
- 通过执行命令`set low_priority_updates=1`，使该连接发出的更新请求优先级降低。
- 通过指定`INSERT`、`UPDATE`、`DELETE`语句的`LOW_PRIORITY`属性，降低该语句的优先级。

折中方案：

MySQL提供了系统参数`max_write_lock_count`来调节读写冲突，为此系统参数设置合适的值，当一个表的读锁达到这个值后，MySQL就暂时将写请求的优先级降低，给读进程一定获得锁的机会。

## 3、InnoDB锁问题

准备一张表

```mysql
CREATE TABLE `user`(
 id INT UNSIGNED AUTO_INCREMENT,
 username VARCHAR(32) NOT NULL COMMENT '用户名',
 nickname VARCHAR(64) NOT NULL COMMENT '昵称',
 PRIMARY KEY(id)
) ENGINE=INNODB
```

准备一些数据

```mysql
INSERT INTO `user` (username,nickname) VALUES  ('a123456','Jackson'),('b296007','Diana'),('d785415','Tom');
```

### 3.2、获取InnoDB行锁争用情况

可以通过检查`innodb_row_lock`状态变量来分析系统上的行锁的争夺情况：

```mysql
mysql> show status like 'innodb_row_lock%';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 0     |
| Innodb_row_lock_time_avg      | 0     |
| Innodb_row_lock_time_max      | 0     |
| Innodb_row_lock_waits         | 0     |
+-------------------------------+-------+
5 rows in set (0.00 sec)
```

如果发现锁争用比较严重，如`Innodb_row_lock_waits`和`Innodb_row_lock_time_avg`的值比较高。

可以通过设置InnoDB Monitors来进一步观察发生锁冲突的表、数据行等，并分析锁争用的原因：

### 3.3、InnoDB的行锁模式及加锁方法

InnoDB实现了一下两种类型的行锁：

- 共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排它锁。
- 排它锁（X）：允许获得排他锁的事物更新数据，阻止其他事务取得相同的是数据集的共享读锁和排他写锁。

为了允许行锁和表锁共存，实现多粒度锁机制，InnoDB还有两种内部使用的意向锁（Intention Locks），**这两种意向锁都是表锁**：

- 意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。
- 意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁。

上述锁模式的兼容情况：

|      | X    | IX   | S    | IS   |
| ---- | ---- | ---- | ---- | ---- |
| X    | 冲突 | 冲突 | 冲突 | 冲突 |
| IX   | 冲突 | 兼容 | 冲突 | 兼容 |
| S    | 冲突 | 冲突 | 兼容 | 兼容 |
| IS   | 冲突 | 兼容 | 兼容 | 兼容 |

如果一个事务请求的锁模式与当前的锁兼容，InnoDB就将请求的锁授予该事务；反之，如果两种不兼容，该事务就要等待锁释放。

锁机制有如下特点：

- 意向锁是InnoDB自动加的，不需要用户干预。

- 对于`UPDATE`、`DELETE`和`INSERT`语句，InnoDB会自动给涉及数据集加排它锁（X）；

- 对于普通`SELECT`语句，InnoDB不会加任何锁；

- 事务可以通过以下语句显示的给记录集加共享锁或排他锁：
  - 共享锁（S）：`SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE`。
  - 排它锁（X）：`SELECT * FROM table_name WHERE ... FOR UPDATE`。

------

用户`SELECT ... IN SHARE MODE`获得共享锁，主要是用在数据依存关系时来确认某行记录是否存在，并确保没有人对这个记录进行`UPDATE`和`DELETE`操作。但是如果当前事务也需要对该记录进行更新操作，则很可能造成死锁，对于锁定行记录后需要进行更新操作的应用，应该使用`SELECT ... FOR UPDATE`方式获得排他锁。



***使用`IN SHARE MODE`加锁后再更新记录:***

- 第一步，session 1和session 2都开启事务，并执行查询。

  session 1 and session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where id=1;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

- 第二步，session 1和session 2都通过`lock in share mode`获取共享锁查询。

  session 1 and session 2
  
  ```mysql
  mysql> select * from user where id=1 lock in share mode;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Jackson  |
  +----+----------+----------+
  1 row in set (0.06 sec)
  ```
  
- 第三步，session 1在当前事务进行更新操作，等待锁。

  session 1
  
  ```mysql
  mysql> update user set username='a987654' where id=1;
  等待
  ```
  
  如果一直等待锁，则会等待锁超时。
  
  ```mysql
  mysql> update user set username='a987654' where id=1;
  ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
  
  # 超过锁等待超时; 试着重新启动事务  
  ```
  
- 第四步，session 2也在当前事务进行更新操作，则会导致死锁退出，而session 1则可以获得锁，并更新成功。

  session 2
  
  ```mysql
  mysql> update user set username='a987654' where id=1;
  ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
  
  # 试图锁定时发现死锁; 试着重新启动事务  
  ```
  
  session 1
  
  ```mysql
  mysql> update user set username='a987654' where id=1;
  Query OK, 1 row affected (3.05 sec)
  Rows matched: 1  Changed: 1  Warnings: 0
  ```



***使用`FOR UPDATE`加锁后再更新记录***

- 第一步，session 1和session 2都开启事务，并执行查询

  session 1 and session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where id=1;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

- 第二步，session 1通过`for update`获取排它锁查询

  session 1
  
  ```mysql
  mysql> select * from user where id=1 for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a987654  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```
  
- 第三步，session 2仍然可以查询该记录，但不能对该记录加排它锁

  session 2

  ```mysql
  mysql> select * from user where id=1;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  
  mysql> select * from user where id=1 for update;
  等待
  ```

- 第四步，session 1可以对锁定记录进行更新操作，更新后释放锁

  session 1

  ```mysql
  mysql> update user set username='a987654' where id=1;
  Query OK, 0 rows affected (0.00 sec)
  Rows matched: 1  Changed: 0  Warnings: 0
  
  mysql> commit;
  Query OK, 0 rows affected (0.08 sec)
  ```

  session 2获取锁之后，查询到了session 1的提交记录

  ```mysql
  mysql> select * from user where id=1 for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a987654  | Jackson  |
  +----+----------+----------+
  1 row in set (8.62 sec)
  ```

  

## 4、InnoDB行锁实现方式

InnoDB行锁是通过给索引上的索引项加锁来实现的。InnoDB这种行锁实现特点意味着：**只有通过索引条件检索数据，InnoDB才使用行锁，否则，InnoDB将使用表锁**。

在实际应用中，要特别注意InnoDB行锁的这一特性，不然的话，可能导致大量的锁冲突，从而影响并发性能。

1、在不通过索引条件查询的时候，InnoDB使用的是表锁，而不是行锁。

***InnoDB存储引擎的表在不使用索引时使用表锁***

- 第一步，session 1和session 2都开启事务，并执行查询（注意：session 1和session 2查询的是不同的数据）

  session 1

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where username='a987654';
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a987654  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

  session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where username='b296007';
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  2 | b296007  | Diana    |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

- 第二步，session 1获取排它锁查询

  ```mysql
  mysql> select * from user where username='a987654' for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a987654  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

- 第三步，session 2获取排它锁查询

  ```mysql
  mysql> select * from user where username='b296007' for update;
  等待
  ```

如上面的例子所示，session 1只给`username='a987654'`的行加了排它锁，而session 1是给`username='b296007'`的行加了排它锁，但它却出现了锁等待！原因就是在没有索引的情况下，InnoDB只能使用表锁。

下面我们给username添加索引，再来看看效果。

```mysql
mysql> alter table user add index username_index(username);
Query OK, 0 rows affected (0.89 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

***InnoDB存储引擎的表在使用索引时使用行锁***

- 第一步，session 1和session 2都开启事务，并执行查询（注意：session 1和session 2查询的是不同的数据）

  session 1

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where username='a987654';
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a987654  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

  session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where username='b296007';
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  2 | b296007  | Diana    |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

- 第二步，session 1获取排它锁查询

  ```mysql
  mysql> select * from user where username='a987654' for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a987654  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

- 第三步，session 2获取排它锁查询

  ```mysql
  mysql> select * from user where username='b296007' for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  2 | b296007  | Diana    |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

------

2、由于MySQL的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用了相同的索引键，是会出现锁冲突的。

- 第一步，session 1获取排它锁查询id=1 and nickname='Jackson'行。

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where id=1 and nickname='Jackson' for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a987654  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

- 第二步，session 1获取排它锁查询id=1 and nickname='Diana'行，等待锁。

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where id=1 and nickname='Diana' for update;
  等待
  ```

如上面的例子所示，session 1和session 2查询的是不同的数据行，但id有索引，nickname没有索引，它们使用了相同的索引，所以需要等待锁。

------

3、当表有多个索引的时候，不同的事务可以使用不同的索引锁定不同的行，但不同的事务使用不同的索引不可以锁定相同的行。另外，不论是使用主键索引、唯一索引或普通索引，InnoDB都会使用行锁来对数据加锁。

***不同的事务，不同的索引锁定不同行***

- 第一步，使用id=1的索引获取排它锁

  session 1

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where id=1 for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a987654  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

- 第二步，使用username='b296007'的索引访问记录，因为记录没有被锁定，所以可以获得锁。

  session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where username='b296007' for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  2 | b296007  | Diana    |
  +----+----------+----------+
  1 row in set (0.00 sec)
  
  
  ```

- 第三步，使用username='a987654'的索引访问记录，因为记录被锁定，所以锁等待。

  session 2

  ```mysql
  mysql> select * from user where username='a987654' for update;
  等待
  ```

------

4、即便在条件中使用了索引字段，但是是否使用索引来检查数据是由MySQL通过判断不同执行计划的代价来决定的，如果MySQL认为全表扫描效率更高（比如很小的表），它就不会使用索引，这种情况下InnoDB将使用表锁，而不是行锁。因此，在分析锁冲突时，别忘了检查SQL的执行计划，以确认是否真正使用了索引。

在下面的例子中，检索值的数据类型与索引字段不同，虽然MySQL能够进行数据类型转换，但却不会使用索引，从而导致InnoDB使用表锁。

*检索值的数据类型与索引字段不同:*

```mysql
mysql> explain select * from user where username=1 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: username_index
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 3
     filtered: 33.33
        Extra: Using where
1 row in set, 3 warnings (0.07 sec)

ERROR:
No query specified
```

*检索值的数据类型与索引字段相同:*

```mysql
mysql> explain select * from user where username="1" \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ref
possible_keys: username_index
          key: username_index
      key_len: 130
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)

ERROR:
No query specified
```

如上面的例子所示，当检索值的数据类型与索引字段不同时，扫描了全表（3行）；当检索值的数据类型与索引字段相同时，只扫描了一行，即使用了索引。

## 5、间隙所（Next-Key锁）

当使用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP）”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所有的间隙锁（Kext-Key锁）。

举例来说，假如user表中只有100条数据，其id的值分别是1,2,3...,99,100，SQL如下：

```mysql
mysql> select * from user where id>99;
```

`id>99`是一个范围条件检索，InnoDB不仅会对符合条件的id值为100的记录加锁，也会对id值大于100（这些记录并不存在）的“间隙”加锁。

InnoDB使用间隙所的目的：

1. 防止幻读，以满足相关隔离级别的要求。
2. 满足其恢复和复制的需要。

> Note：在使用范围条件检索并锁定记录时，InnoDB这种加锁机制会阻止符合条件范围内建值的并发插入，这万万会造成严重的锁等待。因此，在实际应用开发中，尤其是并发插入比较多的应用，应尽量优化业务逻辑，尽量使用相等条件来访问更新数据，避免使用范围条件。

Note：InnoDB除了通过范围条件加锁时使用间隙锁外，如果使用相等条件请求一个不存在的记录加锁，InnoDB也会使用间隙锁。

*InnoDB存储引擎的间隙锁阻塞*

- 第一步：分别查询session 1和session 2的隔离级别

  session 1 and session 2

  ```mysql
  mysql> select @@transaction_isolation;
  +-------------------------+
  | @@transaction_isolation |
  +-------------------------+
  | REPEATABLE-READ         |
  +-------------------------+
  1 row in set (0.00 sec)
  ```

- 第二步：session 1对不存在的记录加for update锁

  session 1

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where id=4 for update;
  Empty set (0.00 sec)
  ```

- 第三步：session 2插入id为4的记录（注意：这条记录并不存在），也会出现锁等待；

  session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> insert into user values (4,'a','b');
  等待
  ```

- 第四步：session 1执行rollback；

  session 1

  ```mysql
  mysql> rollback;
  Query OK, 0 rows affected (0.00 sec)
  ```

  由于session 1回退后释放了Next-Key锁，session 2可以获得锁并成功插入记录

  session 2

  ```mysql
  mysql> insert into user values (4,'a','b');
  Query OK, 1 row affected (7.56 sec)
  ```

## 6、恢复和复制的需要，对InnoDB锁机制的影响

## 7、InnoDB在不同的隔离级别下的一致性读及锁的差异

## 8、什么时候使用表锁

对于InnoDB，在绝大部分情况下都应该使用行级锁，因为事务和行锁往往是我们之所以选择InnoDB表的理由。但在个别特殊事务中，也可以考虑使用表级锁。

- 第一种情况是：事务需要更新大部分或全部数据，表又比较大，如果使用默认的行锁，不仅这个事务执行效率低，而且可能造成其他事务长时间锁等待和锁冲突，这种情况下可以考虑使用表锁来提高该事务的执行速度。
- 第二种情况是：事务涉及多个表，比较复杂，很可能引起死锁，造成大量事务回滚。这种情况也可以考虑一次性锁定事务涉及的表，从而避免死锁，减少数据库因事务回滚带来的开销。

当然，应用中这两种事务不能太多，否则，就应该考虑使用MyISAM引擎了。

在InnoDB下，使用表锁要注意一下两点：

1. 使用`LOCK TABLES`虽然可以给InnoDB加表锁，但必须说明的是，表锁不是由InnoDB存储引擎管理的，而是有其上一层——MySQL Server负责，仅当`autocommit=0`、`innodb_table_locks=1`（默认设置）时，InnoDB层才能知道MySQL加的表锁，MySQL Server也才能感知到InnoDB加的行锁，这种情况下，InnoDB才能自动识别涉及的表级锁的死锁；否则InnoDB将无法自动检测并处理这种死锁。

2. 在用`LOCK TABLES`对InnoDB表加锁时要注意，要将`AUTOCOMMIT`设为0，否则MySQL不会给表加锁；事务结束前，不要用`UNLOCK TABLES`释放锁，因为`UNLOCK TABLES`隐含地提交事务；`COMMIT`或`ROLLBACK`并不能释放用`LOCK TABLES`加的表级锁，必须用`UNLOCK TABLES`释放表锁。

   正确的方式：

   ```mysql
   SET autocommit=0;
   LOCK TABLE t1 WRITE,t2 READ,...;
   -- [do something with tables ti and t2 here];
   COMMIT;
   UNLOCK TABLES;
   ```

## 9、关于死锁

产生死锁的原因：

**两个事务都需要获得对方持有的排它锁才能继续完成事务，这种循环锁等待就是典型的死锁。**

InnoDB存储引擎能自动检测到死锁，当检测到死锁时，使一个事务释放锁并回退，另一个事务获得锁，并继续执行。

在涉及外部锁或表锁的情况下，InnoDB并不能完全自动检测到死锁，这需要通过设置锁等待超时参数`innodb_lock_wait_timeout`来解决。需要说明的是，这个参数并不是只用来解决死锁问题的，在并发访问比较高的情况下，如果大量事务因无法立即获得所需的锁而挂起，会占用大量计算机资源，造成严重地性能问题，甚至拖垮数据库。通过设置合适的锁等到超时阈值，可以避免这种情况的发生。

*innodb_lock_wait_timeout（单位秒，默认超时时间50秒）*

```mysql
mysql> show variables like  'innodb_lock_wait_timeout';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_lock_wait_timeout | 50    |
+--------------------------+-------+
1 row in set, 1 warning (0.06 sec)
```

------

通常来说，死锁都是应用设计的问题，通过调整业务流程、数据库对象设计、事务大小，以及访问数据库的SQL语句，绝大部分死锁都是可避免的。

**避免死锁的几种常用方法**

### 9.1、相同逻辑不同存取顺序产生死锁

**在应用中，如果不同的程序会并发存取多个表，应尽量约定以相同的顺序来访问表，这样可以大大降低产生死锁的机会。**

*相同逻辑不同存取顺序产生死锁：*

- 第一步：session 1和session 2都开启事务。

  session 1 and session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  ```

- 第二步：session 1获取id=1的数据的排它锁。

  session 1

  ```mysql
  mysql> select * from user where id=1 for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

- 第三步：session 2插入id=4的数据，获取到排它锁。

  session 2

  ```mysql
  mysql> insert into book values(4,'经济学要义');
  Query OK, 1 row affected (0.00 sec)
  ```

- 第四步：session 1插入id=4的数据，此时该数据排它锁被session 2持有，所以阻塞等待。

  session 1

  ```mysql
  mysql> insert into book values(4,'经济学要义');
  等待
  ```

- 第五步：session 2获取id=1的数据的排它锁，此时id=1的数据的排它锁被session 1持有，此时session 1和session 2相互持有等待对方的锁，因此session 1产生死锁退出，session 2获取到锁查询成功。

  session 2

  ```mysql
  mysql> select * from user where id=1 for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Jackson  |
  +----+----------+----------+
  1 row in set (0.00 sec)
  ```

  session 1

  ```mysql
  mysql> insert into book values(4,'经济学要义');
  ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
  ```

### 9.2、相同逻辑不同顺序产生死锁

**在程序以批量方式处理数据的时候，如果事先对数据排序，保证每个线程按固定的顺序来处理记录，也可以大大降低出现死锁的可能。**

*相同逻辑不同顺序产生死锁：*

- 第一步：session 1和session 2都开启事务。

  session 1 and session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  ```
  
- 第二步：session 1获取id=1的数据的排它锁。

  session 1

  ```mysql
  mysql> select * from user where id=1 for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Jackson  |
  +----+----------+----------+
  1 row in set (0.09 sec)
  ```
  
- 第三步：session 2获取id=2的数据的排它锁。

  session 2

  ```mysql
  mysql> select * from user where id=2 for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  2 | b296007  | Diana    |
  +----+----------+----------+
  1 row in set (3.80 sec)
  ```
  
- 第四步：session 1获取id=2的数据的排它锁，因为id=2的数据的排它锁已经被session 2持有，因此阻塞等待。

  session 1

  ```mysql
  mysql> select * from user where id=2 for update;
  等待
  ```
  
- 第五步：session 2获取id=1的数据的排它锁，因为id=1的数据的排它锁已经被session 1持有，此时session 1和session 2相互等待对方的锁，产生死锁，session 2产生释放锁退出。

  session 2

  ```mysql
  mysql> select * from user where id=1 for update;
  ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
  ```
  
  session 2产生释放锁退出，session 1获得锁，查询成功。
  
  session 1
  
  ```mysql
  mysql> select * from user where id=2 for update;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  2 | b296007  | Diana    |
  +----+----------+----------+
  1 row in set (3.80 sec)
  ```

### 9.3、先申请不够级别的锁产生死锁

**在事务中，如果要更新记录，应该直接申请足够级别的锁，即排他锁，而不应先申请共享锁，更新时再申请排他锁；因为当用户申请排它锁时，其他事务可能又已经获得了相同记录的共享锁，从而造成锁冲突，甚至死锁。**

*先申请不够级别的锁产生死锁：*

- 第一步，session 1和session 2都开启事务。

  session 1 and session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  ```
  
- 第二步，session 1和session 2都通过`lock in share mode`获取共享锁查询

  session 1 and session 2
  
  ```mysql
  mysql> select * from user where id=1 lock in share mode;
  +----+----------+----------+
  | id | username | nickname |
  +----+----------+----------+
  |  1 | a123456  | Jackson  |
  +----+----------+----------+
  1 row in set (0.06 sec)
  ```
  
- 第三步，session 1在当前事务进行更新操作，更新操作需要排它锁，与共享锁是冲突的。此时该数据的共享锁已经被session 2持有， 因此阻塞等待。

  session 1
  
  ```mysql
  mysql> update user set username='a987654' where id=1;
  等待
  ```
  
- 第四步，session 2也在当前事务进行更新操作，则会导致死锁退出，而session 1则可以获得锁，并更新成功。

  session 2
  
  ```mysql
  mysql> update user set username='a987654' where id=1;
  ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
  ```
  
  session 1
  
  ```mysql
  mysql> update user set username='a987654' where id=1;
  Query OK, 1 row affected (3.05 sec)
  Rows matched: 1  Changed: 1  Warnings: 0
  ```

### 9.4、REPEATABLE-READ：空记录判断插入死锁案例

**在`REPEATABLE-READ`隔离级别下，如果两个线程同时对相同条件记录用`select ... for update`加排他锁，在没有符合该条件记录的情况下，两个线程都会加锁成功。程序发现记录尚不存在，就试图插入一条新记录，如果两个线程都这个做，就会出现死锁。这种情况下，将隔离级别改为`READ-COMMITTED`，就可避免问题。**

*空记录判断插入死锁案例：*

- 第一步：session 1和session 2都开启事务。

  session 1 and session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  ```
- 第二步：session 1和session 2都对不存在的记录加排它锁，都可以加锁成功。

  session 1 and session 2

  ```mysql
  mysql> select * from user where id=10 for update;
  Empty set (0.00 sec)
  ```
- 第三步：session 1插入数据，因为session 2已经对该记录加了锁，所以插入操作阻塞。

  session 1

  ```mysql
  mysql> insert into user values (10,'Lisa','Tom');
  等待
  ```

- 第四步：session 2插入数据，因为session 1已经对该记录加了锁，按道理应该阻塞，但因为session 1也在等待session 2释放锁，此时session 1和session 2交叉等待，session 2产生死锁退出。

  session 2

  ```mysql
  mysql> insert into user values (10,'Lisa','Tom');
  ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
  ```
  
  session 2产生死锁退出，session 1获得锁插入成功。
  
  session 1 
  
  ```sql
  mysql> insert into user values (10,'Lisa','Tom');
  Query OK, 1 row affected (26.80 sec)
  ```

### 9.5、READ-COMMITTED：空记录判断插入死锁案例

**当隔离级别为`READ-COMMITTED`时，如果两个线程都先执行`select ... for update`，判断是否存在符合条件的记录，如果没有，就插入记录。此时，只有一个线程能插入成功，另一个线程会出现锁等待，当第1个线程提交后，第2个线程会因主键重出错，虽然这个线程出错了，却会获得一个排他锁！这时如果有第三个线程又来申请排他锁，也会出现死锁。**

对于这种情况，可以直接做插入操作，然后再捕获主键重异常，或者在遇到主键重错误时，总是执行rollback是否获得排它锁。

*空记录判断插入死锁案例：*

- 第一步：设置隔离级别为`READ-COMMITTED`，并重启会话。

  ```mysql
  -- 设置隔离级别为READ-COMMITTED
  mysql> SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
  Query OK, 0 rows affected (0.06 sec)
  
  -- 重启会话，并查询隔离级别
  mysql> SELECT @@transaction_isolation;
  +-------------------------+
  | @@transaction_isolation |
  +-------------------------+
  | READ-COMMITTED          |
  +-------------------------+
  1 row in set (0.00 sec)
  ```

- 第二步：session 1和session 2都开启事务。

  session 1 and session 2

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  ```

- 第三步：session 1和session 2都对不存在的记录加排它锁，都可以加锁成功。

  session 1 and session 2

  ```mysql
  mysql> select * from user where id=10 for update;
  Empty set (0.00 sec)
  ```

- 第四步：session 1插入数据，与`REPEATABLE-READ`隔离级别不同，session 1可以插入成功。

  session 1

  ```mysql
  mysql> insert into user values (10,'Lisa','Tom');
  Query OK, 1 row affected (0.00 sec)
  ```

- 第五步：session 2插入数据，此时数据被session 1锁定，因此阻塞等待。

  session 2

  ```mysql
  mysql> insert into user values (10,'Lisa','Tom');
  等待
  ```

- 第六步：session 1提交事务，session 2获得锁，执行插入，但发现插入记录主键重复，所以抛出异常，但并没有释放排它锁。

  session 1

  ```mysql
  mysql> commit;
  Query OK, 0 rows affected (0.07 sec)
  ```
  
  session 2
  
  ```mysql
  mysql> insert into user values (10,'Lisa','Tom');
  ERROR 1062 (23000): Duplicate entry '10' for key 'user.PRIMARY'
  ```
  
- 第七步：启动session 3，开启事务，并申请排它锁。因为该记录已被session 2锁定，所以需要等待。

  session 3

  ```mysql
  mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select * from user where id=10 for update;
  等待
  ```

- 第八步：session 2对该记录进行更新操作，更新成功，session 3死锁退出。

  ```mysql
  mysql> update user set username='Jack' where id=10;
  Query OK, 1 row affected (0.05 sec)
  Rows matched: 1  Changed: 1  Warnings: 0
  ```

  session 3

  ```mysql
  mysql> select * from user where id=10 for update;
  ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
  ```

### 9.6、查看最后死锁原因

如果出现死锁，可以通过命令`SHOW ENGINE INNODB STATUS`来确定最后一个命令产生的原因。

*`SHOW ENGINE INNODB STATUS`输出样例：*

```mysql
mysql>  SHOW ENGINE INNODB STATUS \G;
*************************** 1. row ***************************
  Type: InnoDB
  Name:
Status:
=====================================
2021-05-31 20:57:51 0x4390 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 10 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 36 srv_active, 0 srv_shutdown, 204389 srv_idle
srv_master_thread log flush and writes: 0
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 2
OS WAIT ARRAY INFO: signal count 2
RW-shared spins 0, rounds 0, OS waits 0
RW-excl spins 1, rounds 30, OS waits 1
RW-sx spins 0, rounds 0, OS waits 0
Spin rounds per wait: 0.00 RW-shared, 30.00 RW-excl, 0.00 RW-sx
------------------------
LATEST DETECTED DEADLOCK
------------------------
2021-05-31 20:36:55 0x624
*** (1) TRANSACTION:
TRANSACTION 127820, ACTIVE 29 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 18, OS thread handle 17296, query id 205 localhost ::1 root statistics
select * from user where id=2 for update

*** (1) HOLDS THE LOCK(S):
RECORD LOCKS space id 1640 page no 4 n bits 72 index PRIMARY of table `jin-yu`.`user` trx id 127820 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 5; compact format; info bits 0
 0: len 4; hex 00000001; asc     ;;
 1: len 6; hex 00000001f319; asc       ;;
 2: len 7; hex 8100000108011d; asc        ;;
 3: len 7; hex 61313233343536; asc a123456;;
 4: len 7; hex 4a61636b736f6e; asc Jackson;;


*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 1640 page no 4 n bits 72 index PRIMARY of table `jin-yu`.`user` trx id 127820 lock_mode X locks rec but not gap waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 5; compact format; info bits 0
 0: len 4; hex 00000002; asc     ;;
 1: len 6; hex 00000001f319; asc       ;;
 2: len 7; hex 8100000108012a; asc       *;;
 3: len 7; hex 62323936303037; asc b296007;;
 4: len 5; hex 4469616e61; asc Diana;;


*** (2) TRANSACTION:
TRANSACTION 127821, ACTIVE 23 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 19, OS thread handle 8708, query id 206 localhost ::1 root statistics
select * from user where id=1 for update

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 1640 page no 4 n bits 72 index PRIMARY of table `jin-yu`.`user` trx id 127821 lock_mode X locks rec but not gap
Record lock, heap no 3 PHYSICAL RECORD: n_fields 5; compact format; info bits 0
 0: len 4; hex 00000002; asc     ;;
 1: len 6; hex 00000001f319; asc       ;;
 2: len 7; hex 8100000108012a; asc       *;;
 3: len 7; hex 62323936303037; asc b296007;;
 4: len 5; hex 4469616e61; asc Diana;;


*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 1640 page no 4 n bits 72 index PRIMARY of table `jin-yu`.`user` trx id 127821 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 5; compact format; info bits 0
 0: len 4; hex 00000001; asc     ;;
 1: len 6; hex 00000001f319; asc       ;;
 2: len 7; hex 8100000108011d; asc        ;;
 3: len 7; hex 61313233343536; asc a123456;;
 4: len 7; hex 4a61636b736f6e; asc Jackson;;

*** WE ROLL BACK TRANSACTION (2)
------------
TRANSACTIONS
------------
Trx id counter 127822
Purge done for trx's n:o < 127820 undo n:o < 0 state: running but idle
History list length 39
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 283603463786616, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 283603463787440, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 283603463784968, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 283603463784144, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 127820, ACTIVE 1285 sec
3 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 18, OS thread handle 17296, query id 210 localhost ::1 root starting
SHOW engine INNODB STATUS
--------
FILE I/O
--------
I/O thread 0 state: wait Windows aio (insert buffer thread)
I/O thread 1 state: wait Windows aio (log thread)
I/O thread 2 state: wait Windows aio (read thread)
I/O thread 3 state: wait Windows aio (read thread)
I/O thread 4 state: wait Windows aio (read thread)
I/O thread 5 state: wait Windows aio (read thread)
I/O thread 6 state: wait Windows aio (write thread)
I/O thread 7 state: wait Windows aio (write thread)
I/O thread 8 state: wait Windows aio (write thread)
I/O thread 9 state: wait Windows aio (write thread)
Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:, sync i/o's:
Pending flushes (fsync) log: 0; buffer pool: 0
1795 OS file reads, 700 OS file writes, 305 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 0.00 writes/s, 0.00 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 0, seg size 2, 0 merges
merged operations:
 insert 0, delete mark 0, delete 0
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 34679, node heap has 1 buffer(s)
Hash table size 34679, node heap has 0 buffer(s)
Hash table size 34679, node heap has 0 buffer(s)
Hash table size 34679, node heap has 0 buffer(s)
Hash table size 34679, node heap has 0 buffer(s)
Hash table size 34679, node heap has 0 buffer(s)
Hash table size 34679, node heap has 1 buffer(s)
Hash table size 34679, node heap has 3 buffer(s)
0.00 hash searches/s, 0.00 non-hash searches/s
---
LOG
---
Log sequence number          305259461
Log buffer assigned up to    305259461
Log buffer completed up to   305259461
Log written up to            305259461
Log flushed up to            305259461
Added dirty pages up to      305259461
Pages flushed up to          305259461
Last checkpoint at           305259461
171 log i/o's done, 0.00 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 137428992
Dictionary memory allocated 395106
Buffer pool size   8192
Free buffers       6911
Database pages     1276
Old database pages 489
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 1128, created 148, written 349
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 1276, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Process ID=8488, Main thread ID=0000000000000658 , state=sleeping
Number of rows inserted 11, updated 2, deleted 3, read 45
0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
Number of system rows inserted 19, updated 327, deleted 7, read 5159
0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
----------------------------
END OF INNODB MONITOR OUTPUT
============================

1 row in set (0.00 sec)

ERROR:
No query specified
```

