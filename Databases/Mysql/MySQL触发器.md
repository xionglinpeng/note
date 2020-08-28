# MySQL触发器

## 一、创建触发器

触发器（trigger）是一个特殊的存储过程，不同的是，执行存储过程要使用`CALL`语句来调用，而触发器的执行不需要使用`CALL`语句来调用，也不需要手工启动，只要当一个预定义的事件发生的时候，就会被MySQL自动调用。

### 1.1、创建只有一个执行语句的触发器

语法：

```mysql
CREATE TRIGGER trigger_name
	trigger_time trigger_event
	ON tbl_name
	FOR EACH ROW
trigger_stamt
```

- `trigger_name`：表示触发器名称，用户自行指定。
- `trigger_time`：表示触发时机，可选值：`BEFORE`、`ALTER`。
- `trigger_event`：表示触发事件，可选值：`INSERT`、`UPDATE`、`DELETE`。
- `tbl_name`：表示建立触发器的表名，即在那张表上建立触发器。
- `trigger_stamt`：表示触发器执行的语句。

示例：

```mysql
CREATE TRIGGER add_user
	BEFORE INSERT
	ON account
	FOR EACH ROW
INSERT INTO user VALUES (1, '小明');
```

### 1.2、创建有多个执行语句的触发器

语法：

```mysql
CREATE TRIGGER trigger_name
	trigger_time trigger_event
	ON tbl_name
	FOR EACH ROW
BEGIN
	trigger_stamt
END
```

创建有多个执行语句的触发器的时候，以`BEGIN`和`END`关键字作为开始何结束，中间包含多条语句。

示例：

```mysql
CREATE TRIGGER add_user
	AFTER INSERT
	ON account
	FOR EACH ROW
BEGIN
	INSERT INTO user VALUES (1, '小明');
	INSERT INTO user VALUES (2, '小王');
	INSERT INTO user VALUES (3, '小张');
END;
```

## 二、查看触发器

### 2.1、使用`SHOW TRIGGERS`语句查看触发器信息

语法：

```mysql
SHOW TRIGGERS;
```

示例：

```mysql

```

字段说明：

- ``：
- ``：
- ``：
- ``：
- ``：
- ``：
- ``：

> 提示
>
> `SHOW TRIGGERS`语句查看当前创建的所有触发器信息，在触发器较少的情况下，使用该语句会很方便。如果要查看特定触发器的信息，可以直接从`information_schema`数据库中的triggers表中查找。

### 2.2、在triggers表中查看触发器信息

在MySQL中，所有触发器的定义都存在`information_schema`数据库的`TRIGGERS`表中，可以通过查询命令SELECTL查看：

```mysql
SELECT * FROM information_schema.TRIGGERS;
```





## 三、删除触发器

语法：

```mysql
DROP TRIGGER [schema_name.]trigger_name;
```

- schema_name表示数据库名称，可选。
- trigger_name表示被删除触发器的名称。



## 四、使用触发器时注意事项

1. 使用触发器的时候需要注意，对于**相同的表，相同触发时机以及相同的事件**，只能创建一个触发器。只要有一个不同，可再次创建。
2. 及时删除不在需要的触发器。触发器定义之后，每次执行触发事件都会激活触发器并执行触发器中的语句。如果需求发生变化，而触发器没有进行相同的改变或者删除，则触发器仍然会执行旧的语句，从而会影响新的数据完整性。因此，要将不再使用的触发器及时删除。









































