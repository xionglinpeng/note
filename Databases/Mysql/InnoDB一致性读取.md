# [InnoDB一致性读取](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)

一致性读取意味着InnoDB使用multi-versioning向查询提供数据库某个时间点的快照。查询将看到在该时间点之前提交的事务所做的更改，而不会看到稍后或未提交的事务所做的更改。此规则的例外是，查询将看到同一事务中以前的语句所做的更改。该例外导致如下异常：如果您更新表中的某些行，`SELECT`会看到更新行的最新版本，但它也可能看到任何行的旧版本。如果其他会话同时更新同一个表，这种例外意味着该表可能处于数据库中从未存在过的状态。

如果事务[隔离级别](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_isolation_level)为[`REPEATABLE READ`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)(默认级别)，那么同一事务中的所有一致读取都将读取由该事务中第一个此类读取创建的快照。通过提交当前事务并在此之后发出新的查询，您可以获得查询的更*(读音:四声)*新的快照。

使用[`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)隔离级别，事务集中的每个一致性读取都读取自己的新快照。

一致性读是InnoDB在[`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)和[`REPEATABLE READ`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)隔离级别中处理`SELECT`语句的默认模式。一致性读不会在它访问的表上设置任何锁，因此，在对表执行一致性读的同时，其他会话可以自由地修改这些表。

假设您正在默认的[`REPEATABLE READ`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)隔离级别中运行。当你发出一个一致性读时(即一个普通的`SELECT`语句)，InnoDB会给你的事务一个时间点，你的查询可以看到数据库这个时间点之前的数据。如果另一个事务删除一行并在给定时间点之后提交，则不会看到该行已被删除。插入和更新的处理方式类似。

> Note
>
> 数据库状态的快照适用于事务中的`SELECT`语句，不一定适用于[DML](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_dml)语句。如果您插入或修改了一些行，然后提交该事务，一个[`DELETE`](https://dev.mysql.com/doc/refman/8.0/en/delete.html)或[`UPDATE`](https://dev.mysql.com/doc/refman/8.0/en/update.html)语句从另一个并发的`REPEATABLE READ`事务发出，可能会影响这些刚刚提交的行，即使会话不能查询它们。如果一个事务更新或删除了另一个事务提交的行，那么这些更改对当前事务是可见的。例如，你可能会遇到如下情况:
>
> ```sql
> SELECT COUNT(c1) FROM t1 WHERE c1 = 'xyz';
> -- Returns 0: 没有行匹配。
> DELETE FROM t1 WHERE c1 = 'xyz';
> -- 删除其他事务最近提交的几行。
> 
> SELECT COUNT(c2) FROM t1 WHERE c2 = 'abc';
> -- Returns 0: 没有行匹配。
> UPDATE t1 SET c2 = 'cba' WHERE c2 = 'abc';
> -- Affects 10 rows: 另一个txn刚刚提交了10行带有“abc”值的数据。
> SELECT COUNT(c2) FROM t1 WHERE c2 = 'cba';
> -- Returns 10: 这个txn现在可以看到它刚刚更新的行。
> ```

您可以通过提交事务，然后执行另一个[`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html)或[`START TRANSACTION WITH CONSISTENT SNAPSHOT`](https://dev.mysql.com/doc/refman/8.0/en/commit.html)来提前您的时间点。

这称为**多版本并发控制（MVCC）**。

在下面的例子中，会话A看到的是B插入的行，只有当B提交了插入，A也提交了，所以时间点比B提交的时间提前了。

```sql
             Session A              Session B

           SET autocommit=0;      SET autocommit=0;
time
|          SELECT * FROM t;
|          empty set
|                                 INSERT INTO t VALUES (1, 2);
|
v          SELECT * FROM t;
           empty set
                                  COMMIT;

           SELECT * FROM t;
           empty set

           COMMIT;

           SELECT * FROM t;
           ---------------------
           |    1    |    2    |
           ---------------------
```

如果你想查看数据库的"最新"状态，可以使用[`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)隔离级别或[锁读](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_locking_read)：

```sql
SELECT * FROM t FOR SHARE;
```

使用[`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)隔离级别，事务集中的每个一致性读取都读取自己的新快照。而对于`FOR SHARE`，则会发生锁读：`SELECT`会阻塞，直到包含最新行的事务结束(参见15.7.2.4节，[锁读](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)))。

一致性读取不能在某些DDL语句上工作：

- 一致性读取不能在[`DROP TABLE`](https://dev.mysql.com/doc/refman/8.0/en/drop-table.html)上工作，因为MySQL不能使用一个已经被删除的表，并且InnoDB会销毁该表。
- 一致性读不能在[`ALTER TABLE`](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html)操作中工作，这些操作会对原始表进行临时拷贝，并在创建临时拷贝时删除原始表。当在事务中重新发出一致性读时，新表中的行是不可见的，因为在获取事务快照时，这些行不存在。在这种情况下，事务返回一个错误：[`ER_TABLE_DEF_CHANGED`](https://dev.mysql.com/doc/mysql-errors/8.0/en/server-error-reference.html#error_er_table_def_changed)，"表定义已更改，请重试事务"

在子句[`INSERT INTO ... SELECT`](https://dev.mysql.com/doc/refman/8.0/en/insert.html)，[`UPDATE ... (SELECT)`](https://dev.mysql.com/doc/refman/8.0/en/update.html)和[`CREATE TABLE ... SELECT`](https://dev.mysql.com/doc/refman/8.0/en/create-table.html)中，对于没有指定`FOR UPDATE`或`FOR SHARE`的查询，读取的类型是不同的：

- 默认情况下，InnoDB对这些语句使用更强的锁，并且[`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html)部分的行为类似于[`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)，在这里，即使是在同一个事务中，每次一致性读时，都会设置并读取自己的新快照。
- 要在这种情况下执行非锁定读，请将事务的隔离级别设置为[`READ UNCOMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted)或[`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)，以避免对从所选表中读取的行设置锁。