# SQL语言：DDL、DML、DQL、DCL

SQL程序语言有四种类型，对数据库的基本操作都属于这四类。它们分别是：数据定义语言（DDL）、数据操纵语言（DML）、数据查询语言（DQL）、数据控制语言（DCL）。

## 数据定义语言（DDL）

DDL全称Data Definition Language，即数据定义语言。定义语言就是定义关系模式、删除关系、修改关系模式以及创建数据库中的各种对象（例如，表、聚簇、索引、视图、函数、存储过程以及触发器等）。

DDL主要由`CREATE`、`ALTER`、`DROP`和`TRUNCATE`等语法组成。

创建表

```sql
CREATE TABLE `user` (
    `id` BIGINT UNSIGNED AUTO_INCREMENT,
    `name` VARCHAR(50) NOT NULL COMMENT '姓名',
    `age` SMALLINT UNSIGNED NOT NULL COMMENT '年龄',
    PRIMARY KEY(id)
)
```

更改表

```sql
ALTER TABLE `user` ADD COLUMN `email` VARCHAR(200) NULL COMMENT '邮箱';
ALTER TABLE `user` DROP COLUMN `email`;
```

清空表数据：

```sql
TRUNCATE TABLE `user`;
```

删除表：

```sql
DROP TABLE `user`;
```

## 数据操纵语言（DML）

DML全称Data Manipulation Language，即数据操纵语言。主要是进行数据的插入、删除和修改操作。

DML主要由`INSERT`、`UPDATE`和`DELETE`等语法组成。

插入数据

```sql
INSERT INTO `user`(`name`,`age`) VALUE ('小月',20);
```

更新数据

```sql
UPDATE `user` SET `age` = 22 WHERE id = 1;
```

删除数据

```sql
DELETE FROM `user` WHERE id = 1;
```

## 数据查询语言（DQL）

DQL全称Data Query Language，即数据查询语言。主要是用来进行数据查询操作的。

DQL主要由`SELECT`语句组成。

查询

```sql
SELECT * FROM `user`;
```

## 数据控制语言（DCL）

DCL全称Data Control Language，即数据控制语言。主要是用来授权或回收访问数据库的某些权限，并控制数据库操作事务发生的时间以及效果，能够对数据库进行监视。

DCL主要授权、取消授权、回滚、提交等操作。

创建用户

```sql
CREATE USER 用户名@地址 IDENTIFIED BY '密码';
-- 创建一个testuser用户，密码111111
eg. CREATE USER testuser@localhost IDENTIFIED BY '111111';
```

给用户授权

```sql
GRANT 权限1, … , 权限n ON 数据库.对象  TO 用户名;
-- 将test数据库中所有对象(表、视图、存储过程，触发器等。*表示所有对象)CREATE,ALTER,DROP,INSERT,UPDATE,DELETE,SELECT赋给testuser用户
GRANT CREATE,ALTER,DROP,INSERT,UPDATE,DELETE,SELECT ON test.* TO testuser@localhost;
```

撤销授权

```sql
REVOKE 权限1, … , 权限n ON 数据库.对象 FORM 用户名;
-- 将test数据库中所有对象的CREATE,ALTER,DROP权限撤销
REVOKE CREATE,ALTER,DROP ON test.* TO testuser@localhost;
```

查看用户权限

```sql
SHOW GRANTS FOR {用户名};
```

删除用户

```sql
DROP USER 用户名;
```

修改用户密码

```sql
USE mysql;
UPDATE USER SET PASSWORD=PASSWORD(密码) WHERE User=用户名 and Host=IP;
FLUSH PRIVILEGES;
```