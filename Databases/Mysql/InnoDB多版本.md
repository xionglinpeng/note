## [InnoDB多版本](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)

InnoDB是一个多版本的存储引擎。它保存关于已更改行的旧版本的信息，以支持并发和回滚等事务特性。这些信息存储在一个称为回滚段的数据结构中的undo表空间中。参见[15.6.3.4节、Undo表空间](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-tablespaces.html)。InnoDB 使用回滚段中的信息来执行事务回滚所需的undo操作。它还使用这些信息来构建行的早期版本，以便进行一致的读取。参见[15.7.2.3、一致非锁定读](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)。

在内部，InnoDB为存储在数据库中的每一行添加了三个字段:

- `DB_TRX_ID` : 6字节，表示插入或更新行的最后一个事务的事务标识符。此外，删除在内部被视为更新，该行中的一个特殊位被设置为已删除。
- `DB_ROLL_PTR` : 7字节，称为滚动指针。滚动指针指向一个写入回滚段的undo日志记录。如果行已更新，则undo日志记录包含在更新行之前重建该行内容所需的信息。
- `DB_ROW_ID` : 6字节，行ID，当插入新的行时，该行ID单调增加。如果InnoDB自动生成聚集索引，索引包含行ID值。否则，DB_ROW_ID列不会出现在任何索引中。

在回滚段中的Undo日志分为insert和update两种。插入undo日志只在事务回滚时需要，事务提交后即可丢弃。更新undo日志也用于一致性读取，但是，只有在没有事务存在的情况下，InnoDB才可以丢弃这些事务，因为InnoDB已经为事务分配了快照，在一致性读取时，需要更新undo日志中的信息来创建一个更早版本的数据库行。有关撤消日志的其他信息，参见[15.6.6节、Undo日志](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-logs.html)。

建议定期提交事务，包括只发出一致读取的事务。否则，InnoDB无法丢弃更新的undo日志中的数据，回滚段可能会变得太大，填满它所在的undo表空间。有关管理undo表空间的信息，参见[15.6.3.4节、Undo表空间](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-tablespaces.html)。

回滚段中的undo日志记录的物理大小通常小于相应插入或更新的行。您可以使用此信息来计算回滚段所需的空间。

在InnoDB多版本模式中，当你用SQL语句删除一行时，它不会立即从数据库中物理删除。InnoDB只有在丢弃为删除而写的update undo log记录时，才会物理地删除相应的行和它的索引记录。这个删除操作称为清除，它非常快，通常与执行删除操作的SQL语句的时间顺序相同。

如果在表中以相同的速度以较小的批处理插入和删除行，则清除线程可能会开始滞后，并且由于所有的"dead"行，表可能会变得越来越大，使所有内容都绑定到磁盘上，速度非常慢。在这种情况下，阻止新的行操作，并通过调优`innodb_max_purge_lag`系统变量为清除线程分配更多的资源。 更多信息，参见[15.8.9节、清除配置](https://dev.mysql.com/doc/refman/8.0/en/innodb-purge-configuration.html)。

### 多版本和二级索引

InnoDB多版本并发控制(MVCC)将二级索引与聚集索引区别对待。聚集索引中的记录是就地更新的，它们隐藏的系统列指向undo日志条目，可以从中重建早期版本的记录。与聚集索引记录不同，二级索引记录不包含隐藏的系统列，也不会就地更新。

当二级索引列更新时，旧的二级索引记录将被标记为删除，新记录将被插入，而标记为删除的记录最终将被清除。当二级索引记录被删除或二级索引页被一个新的事务更新时，InnoDB会在聚集索引中查找数据库记录。在聚集索引中，检查记录的`DB_TRX_ID`，如果记录在读取事务启动后被修改，则从undo日志中检索记录的正确版本。

如果一个二级索引记录被标记为删除，或者二级索引页被一个新的事务更新，[覆盖索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_covering_index)技术是不使用的。InnoDB不是从索引结构中返回值，而是在聚集索引中查找记录。

但是，如果启用了[索引条件下推(ICP)](https://dev.mysql.com/doc/refman/8.0/en/index-condition-pushdown-optimization.html)优化，并且部分`WHERE`条件只能使用来自索引的字段进行计算，MySQL服务器仍然会将这部分`WHERE`条件下推到存储引擎，在存储引擎中使用索引进行计算。如果没有找到匹配的记录，则避免聚集索引查找。如果找到匹配的记录，即使是在删除标记的记录中，InnoDB也会在聚集索引中查找记录。

---

原文：https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html