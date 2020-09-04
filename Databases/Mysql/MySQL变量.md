

MySQL系统变量官方文档：<https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html>



## sql_mode

默认值：

- `ONLY_FULL_GROUP_BY`：对于`GROUP BY`聚合操作，如果在`SELECT`中的列，没有在`GROUP BY`中出现，那么这个SQL是不合法的，因为列不在`GROUP BY`从句中。
- `STRICT_TRANS_TABLES`：在该模式下，如果一个值不能插入到一个事务表中，则中断当前操作，对非事务表不做限制。
- `NO_ZERO_IN_DATE`：在严格模式下，不允许日期和月份为零。
- `NO_ZERO_DATE`：设置该值，MySQL数据库不允许插入零日期，插入零日期会抛出错误而不是警告。
- `ERROR_FOR_DIVISION_BY_ZERO`：在INSERT或UPDATE过程中，如果数据被零除，则产生错误而非警告。如果未给出改模式，那么数据被零除时MySQL返回NULL。
- `NO_ENGINE_SUBSTITUTION`：如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎代替，并抛出一个异常。

其他有效值：

- `NO_AUTO_VALUE_ON_ZERO`：改值影响自增长列的插入。默认设置下，插入`0`或`NULL`代表生成下一个自增长值。如果用户希望插入的值为`0`，而该列又是自增长的，那么这个选项就有用了。
- `PIPES_AS_CONCAT`：将”`||`“视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样的，也和字符串的拼接函数`Concat`类似。
- `ANSI_QUOTES`：启用`ANSI_QUOTES`后，不能用双引号来引用字符串，因为它被解释为识别符。

```
ALLOW_INVALID_DATES
ANSI_QUOTES
ERROR_FOR_DIVISION_BY_ZERO
HIGH_NOT_PRECEDENCE
IGNORE_SPACE
NO_AUTO_VALUE_ON_ZERO
NO_BACKSLASH_ESCAPES
NO_DIR_IN_CREATE
NO_ENGINE_SUBSTITUTION
NO_UNSIGNED_SUBTRACTION
NO_ZERO_DATE
NO_ZERO_IN_DATE
ONLY_FULL_GROUP_BY
PAD_CHAR_TO_FULL_LENGTH
PIPES_AS_CONCAT
REAL_AS_FLOAT
STRICT_ALL_TABLES
STRICT_TRANS_TABLES
TIME_TRUNCATE_FRACTIONAL
```









## 查询变量



## 设置变量



