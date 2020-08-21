# MySQL视图



## 一、视图概述

视图是从一个或者多个表中导出的，视图的行为与表非常相似，但视图是一个虚拟表。在视图中用户可以使用SELECT语句查询数据，以及使用INSERT、UPDATE和DELETE修改记录。从MySQL 5.0开始可以使用视图，视图可以使用户操作方便，而且可以保证数据库系统的安全。

### 1.1、视图的含义

视图时一个虚拟表，是从数据库中一个或多个表中导出来的表。视图还可以从已经存在的视图的基础上定义。

视图一经定义便存储在数据库中，与其相对应的数据并没有像表那样在数据库中再存储一份，通过视图看到的数据只是存放在基本表中的数据。对视图的操作与对表的操作一样，可以对其进行查询、修改和删除。当对通过视图看到的数据进行修改时，相应的基本表的数据也要发送变化；同时，若基本表的数据发生变化，则这种变化也可以自动反映到视图中。

### 1.2、视图的作用

1. 简单化

   看到的就是需要的。视图不仅可以简化用户对数据的理解，也可以简化它们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件。

2. 安全性

   通过视图用户只能查询和修改它们所能见到的数据。数据库中的其他数据则即看不见，也取不到。数据库授权命令可能使每个用户对数据库的检索限制到特性的数据库对象上，但不能授权到数据库特定行和特定的列上。通过视图，用户可以被限制在数据的不同子集上：

   - 使用权限可被限制在基表的行的子集上。
   - 使用权限可被限制在基表的列的子集上。
   - 使用权限可被限制在基表的行和列的子集上。
   - 使用权限可被限制在多个基表的连接所限定的行上。
   - 使用权限可被限制在基表中的数据的统计汇总上。
   - 使用权限可被限制在另一视图的一个子集上，或是一些视图和基表合并后的子集上。

3. 逻辑数据独立性

   视图可帮助用户屏蔽真实表结构变化带来的影响。

## 二、创建视图

### 2.1、创建视图的语法形式

```mysql
CREATE [OR REPLACE] 
	[ALGORITHM = {undefined | merge | temptable}] 
VIEW view_name [(column_list)]
AS select_statement
[WITH [cascaded | local] check option]
```

- `CREATE`：表示创建新的视图
- `REPLACE`：表示替换已经创建的视图。

- `ALGORITHM`
  1. `undefined`：表示MySQL将自动选择算法。
  2. `merge`：表示将使用的视图语句与视图定义合并起来，使得视图定义的某一部分取代语句对应的部分。
  3. `temptable`：表示将视图的结果存储临时表，然后用临时表来执行语句。

- `cascaded`：默认值，表示更新视图时要满足所有相关视图和表的条件。
- `local`：表示更新视图时满足该视图本身定义的条件即可。

### 2.2、在单表上创建视图

创建表student和class表，并初始化一些数据：

```mysql
CREATE TABLE student (
    id BIGINT UNSIGNED,
    name VARCHAR(10) COMMENT '姓名',
    class BIGINT UNSIGNED COMMENT '班级ID',
    yw_score FLOAT(4,2) COMMENT '语文成绩',
    sx_score FLOAT(4,2) COMMENT '数学成绩'
);

CREATE TABLE class (
    id BIGINT UNSIGNED,
    name VARCHAR(10) COMMENT '班级名称'
);

INSERT INTO student values (1,'小明',1,89,92),(2,'小张',2,77,99),(3,'小李',1,91,88);
INSERT INTO class values (1,'一年级一班'),(2,'三年级二班');
```

创建student表视图

```mysql
CREATE VIEW v_student AS SELECT name,yw_score,sx_score,(yw_score + sx_score)/2 FROM student;
```

查询视图

```mysql
select * from v_student;
```

默认情况下，创建视图和基本表的字段是一样的，可以通过如下两种方式设置想要的字段名称：

1. `AS`关键字

   ```mysql
   CREATE VIEW v_student AS 
   SELECT name,
            yw_score,
            sx_score,
            (yw_score + sx_score)/2 AS avg
   FROM student;
   ```

2. `column_list`选项

   ```mysql
   CREATE VIEW v_student(
           name,
           yw_score,
           sx_score,
           avg) AS 
   SELECT name,
           yw_score,
           sx_score,
           (yw_score + sx_score)/2
   FROM student;
   ```

### 2.3、在多表上创建视图

以student和class表创建视图：

```mysql
CREATE VIEW v_student(name,class,yw_score,sx_score,avg) AS
    SELECT s.name,c.name,s.yw_score,s.sx_score,(s.yw_score + s.sx_score)/2 FROM student s LEFT JOIN class c on s.class = c.id ;
```

查询视图

```mysql
select * from v_student;
```

## 三、查看视图

查看视图是查看数据库中已存在的视图的定义。查看视图必须要有`SHOW VIEW`的权限，MySQL数据库下的user表中保存着这个信息。查看视图的方法包括`DESCRIBE`、`SHOW TABLE STATUS`和`SHOW CREATE VIEW`。

### 3.1、使用DESCRIBE语句查看视图基本信息

语法：

```mysql
describe | desc view_name;
```

示例：

```mysql
describe v_student;
```

说明：

总共6个字段，分别如下:

字段名称（Field）、字段类型（Type）、是否为空（Null）、

是否为主/外键（Type）、默认值（Default）、额外信息（Extra）

### 3.2、使用SHOW TABLE STATUS语句查看视图基本信息

语法：

```mysql
show table status like 'view_name' \G;
```

示例：

```mysql
show table status like 'v_student' \G;
```

说明：

Comment列的值为VIEW，说明该表为视图。

其他信息为NULL，说明是一个虚表。

### 3.3、使用SHOW CREATE VIEW语句查看视图详细信息

语法：

```mysql
show create view view_name;
```

示例：

```mysql
show create view v_student;
```

说明：

主要显示了视图的名称和创建视图的SQL语句。

### 3.4、在views表中查看视图详细信息

在MySQL中，information_schema数据库下的views表中存储了所有视图的定义。通过对views表的查询，可以查看数据库中所有视图的详细信息。

SQL如下：

```mysql
select * from information_schema.views;
```













