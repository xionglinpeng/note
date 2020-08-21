# MySQL日志

## 一、 日志简介

MySQL日志主要分4类：

1. 错误日志：记录MySQL服务的启动、运行或停止MySQL服务时出现的问题。
2. 查询日志：记录建立的客户端连接和执行的语句。
3. 二进制日志：记录所有更改数据的语句，可以用于数据复制。
4. 慢查询日志：记录所有执行时间超过`long_query_time`的所有查询或不使用索引的查询。

默认情况下，所有日志创建于MySQL数据目录中。通过刷新日志，可以强制MySQL关闭和重新打开日志文件（或者在某些情况下切换到一个新的日志）。当执行一个`FLUSH LOGS`语句或执行`MySQLadmin flush-logs`或`MySQLadmin refresh`时，将刷新日志。

如果正使用MySQL复制功能，在复制服务器上可以维护更多日志文件，这种日志称为接替日志。

启动日志功能会降低MySQL数据库的性能。例如，在查询非常频繁的MySQL数据库系统中，如果开启了通用查询日志和慢查询日志，MySQL数据库会花费更多的时间记录日志，同时，日志会占用大量的磁盘空间。

## 二、二进制日志



### 1. 启动和设置二进制日志

默认情况下，二进制日志是开启的，可以通过修改MySQL的配置文件来启动和设置二进制日志。

my.ini中，`[MySQLd]`组下面有关于二进制日志的设置：

```
log-bin=[path/to/[filename]]
```

- log-bin：定义开启二进制日志。
- path/to/：表明日志文件所在的目录路径。
- filename：指定了日志文件的名称。
	> 日志文件的名称会以[filename].000001，[filename].000002，的形式递增。

除了日志文件之外，还有一个名称为[filename].index的文件，文件内容为所有日志的
清单，可以使用记事本打开。

expire_logs_days：定义了MySQL清除过期日志的时间，即二进制日志指定删除的天数。默认值为0，表示“没有指定删除”。当MySQL启动或刷新二进制日志时可能删除该文件。

max_binlog_size：定义了单个文件的大小限制，如果二进制日志写入的内容超出给定值，日志就会发生滚动（关闭当前文件，重新打开一个新的日志文件）。不能将该变量设置为大于1GB或小于4096B，默认值是1GB。

如果正在使用大的事物；二进制日志文件大小还可能会超过`max_binlog_size`定义的大小。

查看日志设置：
```mysql
mysql> SHOW VARIABLES LIKE 'log_%';
+----------------------------------------+----------------------------------------+
| Variable_name                          | Value                                  |
+----------------------------------------+----------------------------------------+
| log_bin                                | ON                                     |
| log_bin_basename                       | \program\Mysql\data\binary-log         |
| log_bin_index                          | \program\Mysql\data\binary-log.index   |
| log_bin_trust_function_creators        | OFF                                    |
| log_bin_use_v1_row_events              | OFF                                    |
| log_error                              | \program\Mysql\data\mysql-error.err    |
| log_error_services                     | log_filter_internal; log_sink_internal |
| log_error_suppression_list             |                                        |
| log_error_verbosity                    | 2                                      |
| log_output                             | FILE                                   |
| log_queries_not_using_indexes          | OFF                                    |
| log_raw                                | OFF                                    |
| log_slave_updates                      | ON                                     |
| log_slow_admin_statements              | OFF                                    |
| log_slow_extra                         | OFF                                    |
| log_slow_slave_statements              | OFF                                    |
| log_statements_unsafe_for_binlog       | ON                                     |
| log_throttle_queries_not_using_indexes | 0                                      |
| log_timestamps                         | UTC                                    |
+----------------------------------------+----------------------------------------+
19 rows in set, 1 warning (0.00 sec)
```

> 数据库文件最好不要与日志文件放在同一个磁盘上，这样当数据库文件所在的磁盘发送故障时，可以使用日志文件恢复数据。

### 2. 查看二进制日志

MySQL二进制日志存储了所有的变更信息。

当MySQL创建二进制日志文件时，先创建一个以“filename”为名称，以“.index”为后缀
的文件，再创建一个以“filename”为名称、以“.000001”为后缀的文件。

MySQL服务重新启动一次，以“000001”为后缀的文件就会增加一个，并且后缀名加1递增（执行`flush logs`也会递增该文件）；

如果日志长度超过了`max_binlog_size`的上限（默认是1GB），就会创建一个新的日志文件。

`show bigary logs`语句可以查看当前的二进制日志文件个数以及其文件名。

示例：
```mysql
mysql> show binary logs;
+-------------------+-----------+-----------+
| Log_name          | File_size | Encrypted |
+-------------------+-----------+-----------+
| binary-log.000001 |       156 | No        |
+-------------------+-----------+-----------+
1 row in set (0.00 sec)
```

MySQL二进制日志并不能直接查看，如果要查看日志内容，可以通过`mysqlbinlog`命令查看。

示例：
```mysql
$ mysqlbinlog /program/Mysql/data/binary-log.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#200815 15:37:12 server id 1  end_log_pos 125 CRC32 0x7a22a29e  Start: binlog v 4, server v 8.0.20 created 200815 15:37:12 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
qJA3Xw8BAAAAeQAAAH0AAAABAAQAOC4wLjIwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAACokDdfEwANAAgAAAAABAAEAAAAYQAEGggAAAAICAgCAAAACgoKKioAEjQA
CigBnqIieg==
'/*!*/;
# at 125
#200815 15:37:12 server id 1  end_log_pos 156 CRC32 0x0cbee86b  Previous-GTIDs
# [empty]
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

### 3. 删除二进制日志

#### 3.1 使用`RESET MASTER`语句删错所有二进制日志文件
RESET MASTER语法如下：
```mysql
mysql> reset master;
Query OK, 0 rows affected (0.30 sec)
```
执行完该语句后，所有二进制日志将被删除，MySQL会重新创建二进制日志，新的日志文件
扩展名将重新从000001开始编号。

#### 3.2 使用`PURGE MASTER LOGS`语句删除指定日志文件


1. 删除文件名编号比指定文件名编号小的所有日志文件

	首先执行4次`flush logs`，以生成4个二进制日志文件：
	
	```mysql
	mysql> flush logs;
	Query OK, 0 rows affected (0.24 sec)

	mysql> flush logs;
	Query OK, 0 rows affected (0.30 sec)

	mysql> flush logs;
	Query OK, 0 rows affected (0.28 sec)

	mysql> flush logs;
	Query OK, 0 rows affected (0.21 sec)
	```
	使用`show binary logs;`查看二进制文件数量：
	```mysql
	mysql> show binary logs;
	+---------------+-----------+-----------+
	| Log_name      | File_size | Encrypted |
	+---------------+-----------+-----------+
	| binlog.000001 |       200 | No        |
	| binlog.000002 |       200 | No        |
	| binlog.000003 |       200 | No        |
	| binlog.000004 |       200 | No        |
	| binlog.000005 |       156 | No        |
	+---------------+-----------+-----------+
	5 rows in set (0.00 sec)
	```
	删除`binlog.000004`文件编号之前的二进制日志文件：
	```mysql
	mysql> purge master logs to 'binlog.000004';
	Query OK, 0 rows affected (0.13 sec)
	```
2. 删除指定日期以前的所有日志文件
	
	```mysql
	mysql> reset master;
	Query OK, 0 rows affected (0.15 sec)
	
	mysql> flush logs;
	Query OK, 0 rows affected (0.30 sec)
	
	mysql> flush logs;
	Query OK, 0 rows affected (0.17 sec)
	
	mysql> show binary logs;
	+---------------+-----------+-----------+
	| Log_name      | File_size | Encrypted |
	+---------------+-----------+-----------+
	| binlog.000001 |       200 | No        |
	| binlog.000002 |       200 | No        |
	| binlog.000003 |       156 | No        |
	+---------------+-----------+-----------+
	3 rows in set (0.00 sec)
	
	mysql> purge master logs before '20200815';
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> show binary logs;
	+---------------+-----------+-----------+
	| Log_name      | File_size | Encrypted |
	+---------------+-----------+-----------+
	| binlog.000001 |       200 | No        |
	| binlog.000002 |       200 | No        |
	| binlog.000003 |       156 | No        |
	+---------------+-----------+-----------+
	3 rows in set (0.00 sec)
	
	mysql> purge master logs before '20200816';
	Query OK, 0 rows affected, 1 warning (0.16 sec)
	
	mysql> show binary logs;
	+---------------+-----------+-----------+
	| Log_name      | File_size | Encrypted |
	+---------------+-----------+-----------+
	| binlog.000003 |       156 | No        |
	+---------------+-----------+-----------+
	1 row in set (0.00 sec)
	```

## 三、错误日志

错误日志文件包含了当MySQLd启动和停止时，以及服务器在运行过程中发生任何严重错误
时的相关信息。

### 1. 启动和设置错误日志

默认情况下，错误日志会记录到数据库的数据目录下。如果没有在配置文件中指定文件名，
则文件名默认为`hostname.err`。如果执行了`FLUSH LOGS`，错误日志会重新加载。

错误日志的启动和停止以及指定日志文件名都可以通过修改my.ini（或者my.cnf）来
配置。错误日志的配置项是`log-error`。在`[mysqld]`下配置`log-error`，则启动
错误日志。如果需要指定文件名，则配置项如下：
```
[mysqld]
log-error=[path/to/[file_name]]
```
配置完成之后，需要重启MySQLf服务使其生效。

### 2. 查看错误日志

通过错误日志可以监视系统的运行状态、便于即使发现故障、修复故障。MySQL错误日志
是以文本文件的形式存储的，可以使用文件编辑器直接查看MySQL错误日志。

查询错误日志的存储路径：
```mysql
mysql> SHOW VARIABLES LIKE 'log_error';
+---------------+-------------------------------------+
| Variable_name | Value                               |
+---------------+-------------------------------------+
| log_error     | \program\Mysql\data\mysql-error.err |
+---------------+-------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

### 3. 删除错误日志

MySQL的错误日志是以文本文件的形式存储在文件系统中的，可以直接删除。

在运行状态下删除错误日志文件后，MySQL并不会自动创建日志文件。`flush logs`在
重新加载日志的时候，如果文件不存在，则会自动创建。所以在删除错误日志之后，如果
需要重建日志文件，需要在服务器端执行以下命令：
```shell
$ mysqladmin -uroot -p[password] flush-logs
```
或者客户端登录mysql数据库执行`flush logs`语句：
```mysql
mysql> flush logs;
Query OK, 0 rows affected (0.52 sec)
```

> 对于MySQL5.5.7以前的版本，`flush logs`可以将错误日志文件重命名为filename.err.old，
> 并创建新的日志文件。但是从MySQL5.5.7开始，`flush logs`只是重新打开日志文件，
> 并不做日志备份和创建操作。