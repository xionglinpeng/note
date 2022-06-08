# [13.3.7 MySQL XA事务](https://dev.mysql.com/doc/refman/5.7/en/xa.html)

[`InnoDB`](https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)存储引擎支持XA事务。MySQL XA的实现是基于X/Open CAE文档*Distributed Transaction Processing: The XA Specification*。这个文档是由Open Group发布，可在http://www.opengroup.org/public/pubs/catalog/c193.htm上获取。当前XA实现的限制在[第13.3.7.3节, “XA事务的限制”](https://dev.mysql.com/doc/refman/5.7/en/xa-restrictions.html)进行描述。

在客户端，没有特殊的要求。MySQL服务器的XA接口由以`XA`关键字开头的SQL语句组成。MySQL客户端程序必须能够发送SQL语句并理解XA语句接口的语义。它们不需要连接到最近的客户端库。旧的客户端库也可以工作。

在MySQL连接器中，MySQL Connector/J 5.0.0以及更高版本直接支持XA，通过类接口为你处理XA SQL语句接口。

XA支持分布式事务，即允许多个独立的事务资源参与全局事务的能力。事务性资源通常是RDBMS，也可能是其他类型的资源。

全局事务包含几个本身是事务性的操作，但是所有的操作必须作为一个整体成功完成，或者作为一个整体全部回滚。本质上，这将ACID属性“提升了一个层次”，以便多个ACID事务可以作为同样具有ACID属性的全局操作的组件一起执行。（与非分布式事务一样，如果你的应用程序对读取现象敏感，[`SERIALIZABLE`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_serializable)可能是首选项。[`REPEATABLE READ`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)对于分布式事务可能不够。）

一些分布式事务的例子：

- 应用程序可以作为集成工具，将消息服务与RDBMS组合在一起。应用程序确保事务处理消息的发送、检索和处理，也包含事务数据库，都在全局事务中发生。你可以把它想象成“事务性的邮件”。
- 应用程序执行涉及不同数据库服务器的操作，如MySQL服务器和Oracle服务器(或多个MySQL服务器)。其中涉及多个服务器的操作必须作为全局事务的一部分发生，而不是作为每个服务器本地的独立事务发生。
- 银行在RDBMS中保存帐户信息，并通过自动柜员机(ATM)分发和接收资金。有必要确保ATM操作正确地反映在帐户中，但是仅使用RDBMS不能做到这一点。全局事务管理器集成了ATM和数据库资源，以确保金融事务的整体一致性。

使用全局事务的应用程序包含一个或多个资源管理器和一个事务管理器：

- 资源管理器(RM)提供对事务性资源的访问。数据库服务器是一种资源管理器。它必须能够提交或回滚由RM管理的事务。
- 事务管理器(TM)作为全局事务的一部分协调事务。它与每个处理这些事务的RM通信。全局事务中的各个事务是全局事务的“分支”。全局事务及其分支由稍后描述的命名方案标识。

XA的MySQL实现使MySQL服务器能够充当在全局事务中处理XA事务的资源管理器。连接到MySQL服务器的客户端程序充当事务管理器。

要执行全局事务，需要知道包含哪些组件，并使每个组件达到可以提交或回滚的时间点。根据每个组件对其成功能力的报告，它们必须都作为原子组提交或回滚。也就是说，要么所有组件都必须提交，要么所有组件都必须回滚。要管理全局事务，必须考虑到任何组件或连接的网络都可能失败。

执行全局事务的过程使用两阶段提交(2PC)。这发生在执行了全局事务的分支执行的操作之后。

1. 在第一阶段，准备好所有分支。也就是说，TM告诉他们准备好去提交。通常，这意味着每个管理分支的RM在稳定的存储中记录分支的操作。分支表明它们是否能够做到这一点，这些结果将用于第二阶段。
2. 在第二阶段，TM告诉RM是提交还是回滚。如果所有分支都表明它们准备好可以提交，那么所有分支都将被告知提交。如果有任何一个分支在它准备好时表明不能提交，那么所有分支都将被告知回滚。

在某些情况下，全局事务可能使用一阶段提交（1PC）。例如，当事务管理器发现全局事务只包含一个事务性资源（即单个分支）时，可以通知该资源同时准备和提交。

## [13.3.7.1 XA事务SQL语法](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)

要在MySQL中执行XA事务，请使用以下语句：

```sql
XA {START|BEGIN} xid [JOIN|RESUME]

XA END xid [SUSPEND [FOR MIGRATE]]

XA PREPARE xid

XA COMMIT xid [ONE PHASE]

XA ROLLBACK xid

XA RECOVER [CONVERT XID]
```

对于[`XA START`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)，可以识别`JOIN`和`RESUME`子句，但没有效果。

对于[`XA END`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)，可以识别`SUSPEND [FOR MIGRATE]`子句，但没有效果。

每个XA语句以`XA`关键字开始，其中大多数语句都需要一个*`xid`*值。*`xid`*是XA事务标识符。它指示该语句应用于那个事务。*`xid`*值由客户端提供，或者由MySQL服务器生成。一个*`xid`*值由一到三部分组成：

```none
xid: gtrid [, bqual [, formatID ]]
```

*`gtrid`*是全局事务标识符，*`bqual`*是分支限定符，*`formatID`*是一个数字，用于标识*`gtrid`*和*`bqual`*的值所使用的格式。 如语法所示，*`gtrid`*和*`bqual`*是可选项。如果没有给出，*`bqual`*的值默认为`''`，*`formatID`*的值默认是1。

*`gtrid`*和*`bqual`*必须是字符串字面量，每个字符串长度不超过64字节（不是字符）。*`gtrid`*和*`bqual`*可以用几种方式指定。你可以使用带引号的字符串(`'ab'`)，十六进制字符串(`X'6162'`, `0x6162`)，或bit值(`b'nnnn'`)。

*`formatID`*是一个无符号整数。

*`gtrid`*和*`bqual`*的值由MySQL服务器底层XA支持的例程以字节解释。但是，在解析包含XA语句的SQL语句时，服务器使用一些特定的字符集。为了安全起见，*`gtrid`*和*`bqual`*写出十六进制字符串。

*`xid`*的值通常由事务管理器生成。一个TM生成的值必须与其他TM生成的值不同。给定的TM必须能够在[`XA RECOVER`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)语句返回的值列表中识别自己的*`xid`*值。

[`XA START xid`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)使用给定的*`xid`*值启动XA事务。每个XA事务必须有一个唯一的*`xid`*值，因此这个值在当前不能被另一个XA事务使用。唯一性使用*`gtrid`*和*`bqual`*的值进行评判。  XA事务的所有以下XA语句必须指定使用在[`XA START`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)语句中给定的相同的*`xid`*值。如果你使用这些语句中的任何一条，但指定的*`xid`*值与某些现有的XA事务不对应，那么将会发生错误。

一个或多个XA事务可以是相同全局事务的一部分。在给定全局事务中所有XA事务的*`xid`*值必须使用相同的*`gtrid`*值。因此，*`gtrid`*值必须全局唯一，这样就不会对给定XA事务所属的全局事务产生歧义。在给定全局事务中的每个XA事务，*`xid`*值的*`bqual`*部分必须不同。（要求*`bqual`*值不同是当前MySQL XA实现的一个限制。它不是XA规范的一部分。）

[`XA RECOVER`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)语句返回MySQL服务器上处于`PREPARED`状态的XA事务信息。（参见[13.3.7.2节,“XA事务状态”](https://dev.mysql.com/doc/refman/5.7/en/xa-states.html)）输出的每一行都是包含在服务器上的一个此类XA事务，无论是哪个客户端启动了它。

[`XA RECOVER`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)输出行如下所示（例如*`xid`*值由`'abc'`,`'def'`和 `7`三部分组成）：

```sql
mysql> XA RECOVER;
+----------+--------------+--------------+--------+
| formatID | gtrid_length | bqual_length | data   |
+----------+--------------+--------------+--------+
|        7 |            3 |            3 | abcdef |
+----------+--------------+--------------+--------+
```

字段的含义如下：

- `formatID`是事务*`xid`*的*`formatID`*部分。
- `gtrid_length`是*`xid`*的*`gtrid`*部分的字节长度。
- `bqual_length`*是`xid`*的*`bqual`*部分的字节长度。
- `data`是*`xid`*的*`gtrid`*和*`bqual`*部分的拼接。

XID值可能包含不可打印的字符。在MySQL 5.7.5中，[`XA RECOVER`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)允许一个可选的`CONVERT XID`子句，这可以使客户端获取十六进制的XID值。

## [13.3.7.2 XA事务状态](https://dev.mysql.com/doc/refman/5.7/en/xa-states.html)

XA事务通过以下状态进行：

1. 使用[`XA START`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)启动XA事务，并将其置于`ACTIVE`状态。
2. 对于`ACTIVE`状态的XA事务，执行该事务的DML语句，然后使用[`XA END`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)语句，将事务置于`IDLE`状态。
3. 对于`IDLE`状态的XA事务，你可以使用[`XA PREPARE`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)语句或一个`XA COMMIT ... ONE PHASE`语句：
   - [`XA PREPARE`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)将事务置于`PREPARED`状态。此时，[`XA RECOVER`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)语句在其输出中包含*`xid`*的值，因为[`XA RECOVER`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)列出了所有处于`PREPARED`状态的事务。
   - `XA COMMIT ... ONE PHASE`准备并提交事务，[`XA RECOVER`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)没有列出*`xid`*的值，因为事务已经终止了。
4. 对于`PREPARED`状态的XA事务，你可以使用[`XA COMMIT`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)语句提交并终止事务，或者[`XA ROLLBACK`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)回滚或终止事务。

下面是一个简单的XA事务，它将一行插入到表中，作为全局事务的一部分：

```sql
mysql> XA START 'xatest';
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO mytable (i) VALUES(10);
Query OK, 1 row affected (0.04 sec)

mysql> XA END 'xatest';
Query OK, 0 rows affected (0.00 sec)

mysql> XA PREPARE 'xatest';
Query OK, 0 rows affected (0.00 sec)

mysql> XA COMMIT 'xatest';
Query OK, 0 rows affected (0.00 sec)
```

在给定的客户端连接上下文中，XA事务和本地（非XA）事务是互斥的。例如，如果已经使用[`XA START`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)开始XA事务，则在提交或回滚XA事务之前不能启动本地事务。相反，如果使用[`START TRANSACTION`](https://dev.mysql.com/doc/refman/5.7/en/commit.html)启动本地事务，则在提交或回滚事务之前，不能使用XA语句。

如果XA事务处于`ACTIVE`状态，则不能使用任何导致隐式提交的语句。这将违反XA的约定，因为你不能回滚XA事务。如果你尝试执行这样的语句，将会引发以下错误：

```none
ERROR 1399 (XAE07): XAER_RMFAIL: The command cannot be executed
when global transaction is in the ACTIVE state
```

适用上述注释的语句列在[章节13.3.3，“导致隐式提交的语句”](https://dev.mysql.com/doc/refman/5.7/en/implicit-commit.html)中。

## [13.3.7.3 XA事务的限制](https://dev.mysql.com/doc/refman/5.7/en/xa-restrictions.html)

对XA事务的支持仅限于`InnoDB`存储引擎。

对于“外部XA”，MySQL服务器担任资源管理器，客户端程序担任事务管理器。对于“内部XA”，MySQL服务器内的存储引擎担任RM，服务器本身担任TM。内部XA的支持受单个存储引擎的功能限制。内部XA需要处理包含多个存储引擎的XA事务。内部XA的实现要求存储引擎支持表处理级别的两阶段提交，当前这只适用于`InnoDB`。

对于[`XA START`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)，可以识别`JOIN`和`RESUME`子句，但没有效果。

对于[`XA END`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)，可以识别`SUSPEND [FOR MIGRATE]`子句，但没有效果。

对于全局事务中的每个XA事务，*`xid`*值的*`bqual`*部分必须是不同的，这是当前MySQL XA实现的一个限制。它不是XA规范的一部分。

在MySQL 5.7.7之前，XA事务与复制(replication)不兼容。这是因为处于`PREPARED`状态的XA事务会在完全关闭服务器或客户端断开连接时回滚。同样，处于`PREPARED`状态的XA事务在服务器异常关闭然后再次启动的情况下仍然处于`PREPARED`状态，但是不能将事务的写入二进制日志。在这两种情况下，都不能正确地复制XA事务。

在MySQL 5.7.7及以后版本中，行为发生了变化，XA事务分为两部分写入二进制日志。当发出`XA PREPARE `时，事务的第一部分直到`XA PREPARE `使用初始GTID写入。`XA_prepare_log_event`用于标识二进制日志中的此类事务。当发出`XA COMMIT`或`XA ROLLBACK`时，事务的第二部分仅包含`XA COMMIT`或`XA ROLLBACK`语句，将使用第二个GTID写入。请注意，事务的初始部分(由`XA_prepare_log_event`标识)后面不一定跟着它的`XA COMMIT`或`XA ROLLBACK`，这可能会导致任意两个XA事务的二进制日志交叉记录。XA事务的两个部分甚至可以出现在不同的二进制日志文件中。这意味着处于`PREPARED`状态的XA事务现在是持久的，直到发出显式的`XA COMMIT`或`XA ROLLBACK`语句，以确保XA事务与复制(replication)兼容。

在副本(replica)上，在准备好XA事务之后，立即将其与副本(replica)应用程序线程分离，副本上的任何线程都可以提交或回滚该事务。这意味着相同的XA事务可以在[`events_transactions_current`](https://dev.mysql.com/doc/refman/5.7/en/performance-schema-events-transactions-current-table.html)表中以不同线程上的不同状态出现。[`events_transactions_current`](https://dev.mysql.com/doc/refman/5.7/en/performance-schema-events-transactions-current-table.html)表显示线程上最近监视的事务事件的当前状态，当线程空闲时不更新此状态。因此，在被另一个线程处理之后，XA事务仍然可以显示为原始应用程序线程的`PREPARED`状态。要确定仍然处于`PREPARED`状态且需要恢复的XA事务，请使用[`XA RECOVER`](https://dev.mysql.com/doc/refman/5.7/en/xa-statements.html)语句，而不是使用Performance Schema事务表。

在MySQL 5.7.7及更高版本中，使用XA事务存在以下限制:

- XA事务对于二进制日志的意外停止没有完全弹性。如果服务器在执行`XA PREPARE`、`XA COMMIT`、`XA ROLLBACK`或`XA COMMIT ... ONE PHASE`语句时发生意外暂停，服务器可能无法恢复到正确的状态，使服务器和二进制日志处于不一致的状态。在这种情况下，二进制日志可能包含未应用的额外XA事务，或者错过已应用的XA事务。此外，如果启用了GTID，在恢复`@@GLOBAL.GTID_EXECUTED`之后可能无法正确描述已应用的事务。请注意，如果在`XA PREPARE`之前、在`XA PREPARE`和`XA COMMIT`(或`XA ROLLBACK`)之间、或在`XA COMMIT`(或`XA ROLLBACK`)之后发生意外停机，服务器和二进制日志将被正确恢复并恢复到一致状态。

- 不支持将复制(replication)过滤器或二进制日志过滤器与XA事务结合使用。对表进行筛选可能会导致副本上的XA事务为空，并且不支持空XA事务。同时，在一个副本(replication)上设置[`master_info_repository=TABLE`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-replica.html#sysvar_master_info_repository)和[`relay_log_info_repository=TABLE`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-replica.html#sysvar_relay_log_info_repository)，这成为在MySQL 8.0上的默认值，数据引擎事务的内部状态在经过筛选的XA事务之后更改，并且可能与复制(replication)事务上下文状态一致。

  每当XA事务受到复制(replication)筛选器影响时，无论事务是否为空，都会记录错误[`ER_XA_REPLICATION_FILTERS`](https://dev.mysql.com/doc/mysql-errors/5.7/en/server-error-reference.html#error_er_xa_replication_filters)。如果事务不是空的，那么副本可以继续运行，但是您应该采取步骤停止对XA事务使用复制过滤器，以避免潜在的问题。如果事务为空，则复制停止。在这种情况下，副本可能处于未确定状态，在这种状态下复制进程的一致性可能会受到影响。特别是，在该副本的副本上设置的`gtid_executed`可能与源上的不一致。要解决这种情况，请隔离源并停止所有复制，然后在整个复制拓扑中检查GTID一致性。撤消生成错误消息的XA事务，然后重新启动复制。

- 在MySQL 5.7.19之前，[`FLUSH TABLES WITH READ LOCK`](https://dev.mysql.com/doc/refman/5.7/en/flush.html#flush-tables-with-read-lock)不兼容XA事务。

- XA事务对于基于语句的复制是不安全的。如果在源上并行提交的两个XA事务在副本上按相反的顺序准备，则可能会发生无法安全解析的锁定依赖关系，并且复制可能会因为副本上的死锁而失败。这种情况可能发生在单线程或多线程副本中。当设置[`binlog_format=STATEMENT`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_binlog_format)时，会对XA事务中的DML语句发出警告。当[`binlog_format=MIXED`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_binlog_format)或[`binlog_format=ROW`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_binlog_format)时, DML语句在XA事务记录使用基于行的复制，潜在的问题并不存在。