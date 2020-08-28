# MySQL视图



## 一、视图概述

视图是从一个或者多个表中导出的，视图的行为与表非常相似，但视图是一个虚拟表。在视图中用户可以使用SELECT语句查询数据，以及使用INSERT、UPDATE和DELETE修改记录。从MySQL 5.0开始可以使用视图，视图可以使用户操作方便，而且可以保证数据库系统的安全。

### 1.1、视图的含义

视图是一个虚拟表，是从数据库中一个或多个表中导出来的表。视图还可以从已经存在的视图的基础上定义。

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

## 四、修改视图

### 4.1、使用CREATE OR REPLACE VIEW语句修改视图

语法：

```mysql
CREATE [OR REPLACE] 
	[ALGORITHM = {undefined | merge | temptable}] 
VIEW view_name [(column_list)]
AS select_statement
[WITH [cascaded | local] check option]
```

修改视图的语句与创建视图的语句是完全一样的：

- 当视图已经存在时，修改语句对视图进行修改；
- 当视图不存在时，则创建视图；

### 4.2、使用ALTER语句修改视图

语法：

```mysql
ALTER [ALGORITHM = {undefined | merge | temptable}] 
VIEW view_name [(column_list)]
AS select_statement
[WITH [cascaded | local] check option]
```

`ALTER`语句与`CREATE OR REPLACE VIEW`语句大同小异，只是它只能修改，不能在视图不存在时，创建视图。

## 五、更新视图

更新视图是指通过视图来插入、更新、删除表中的数据。因为视图是一个虚拟表，其中没有数据，所以通过视图更新的时候都是转到基本表上进行更新的，如果对视图增加或者删除记录，实际上是对其基本表增加或者删除记录。

对视图的更新分别对应：`INSERT`、`UPDATE`、`DELETE`。

当视图中包含有如下内容时，视图的更新操作将不能被执行：

1. 视图中不包含基表中被定义为非空的列。
2. 在定义视图的SELECT语句后的字段列表中使用了数学表达式。
3. 在定义视图的SELECT语句后的字段列表中使用聚合函数。
4. 在定义视图的SELECT语句中使用了`DISTINCT`、`UNION`、`TOP`、`GROUP BY`或`HAVING`子句。

## 六、删除视图

语法：

```mysql
DROP VIEW [IF EXISTS]
	view_name [, view_name] ...
	[RESTRICT | CASCADE]
```

view_name是要删除的视图名称，可以添加多个需要删除的视图名称，各个名称之间使用逗号分隔开。删除视图必须拥有DROP权限。

## 七、总结信息

### 7.1、视图与表的区别

1. 视图是已经编译好的SQL语句，是基于SQL语句结果集的可视化表，而表不是。
2. 视图没有实际的物理记录，而表有。
3. 表是内容，视图是窗口。
4. 表占用物理空间，而视图不占用物理空间。视图只是逻辑感念的存在，表可以及时修改，但视图只能用创建的语句来修改。
5. 视图是查看数据表的一种方法，可以查询数据表中某些字段构成的数据，只是一些SQL语句的集合。从安全的角度来说，视图可以防止用户接触数据表，因而用户不知道表结构。
6. 表属于全局模式中的表，是实表；视图属于局部模式的表，是虚表。
7. 视图的建立和删除只影响视图本身，不影响对应的基本表。

### 7.2、视图与表的联系

视图（VIEW）是在基本表之上建立的表，它的结构（所定义的列）和内容（所有的记录）都来自基本表，它依据基本表存在而存在。一个视图可以对应一个基本表，也可以对应多个基本表。视图是基本表的抽象和在逻辑意义上建立的新关系。

### 7.3、视图与表的性能

视图的好处很明显：

- 简化复杂查询

- 安全控制

  限制特定用户的数据访问。

- 计算列

  视图的某一列字段是通过计算得出的。

- 向后兼容

  例如重构数据库以适应新的业务需求，对原有的业务代码可以使用视图兼容。

但是**视图的性能差于对基本表的直接查询**，这是因为基本表才有内容，视图仅仅是对基本表的一个窗口而已，对视图进行查询的时候，最终还是要转到基本表上进行查询。



















