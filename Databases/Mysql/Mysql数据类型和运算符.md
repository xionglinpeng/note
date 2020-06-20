# Mysql数据类型和运算符





## Mysql数据类型



#### DATE

DATE类型用在仅需日期值时，没有时间部分，在存储时需要3字节。日期格式为`'YYYY-MM-DD'`。其中`YYYY`表示年，`MM`表示月，`DD`表示日。在给DATE类型的字段赋值时，可以使用**字符串类型**或者**数字类型**的数据插入，只要符合DATE的日期格式即可。

1. 以`'YYYY-MM-DD'`或者`'YYYYMMDD'`字符串格式表示的日期，取值范围为`'1000-01-01'` ~ `'9999-12-3'`。
2. 以`'YY-MM-DD'`或者`'YYMMDD'`字符串格式表示的日期，在这里YY表示两位年值。包含两位年值的日期会令人模糊，因为不知道世纪。MySQL使用一下规则解释两位年值：
   - `'00 ~ 69'`范围的年值转换为`'2000 ~ 2069'`；
   - `'70 ~ 99'`范围的年值转换为`'1970 ~ 1999'`；
3. 使用`CURRENT_DATE`或者`NOW()`，插入当前系统日期。

> MySQL允许“不严格”语法：任何标点符号都可以用作日期部分之间的间隔符。例如：'98-11-31'、'98.11.31'、'98/11/31'和'98@11@31'是等价的，这些值也可以正确的插入到数据库中。

#### DATETIME

DATETIME类型用于需要同时包含日期和时间信息的值，在存储时需要8字节。日期格式为'YYYY-MM-DD HH:MM:SS'，其中YYYY表示年，MM表示月，DD表示日，HH表示小时，MM表示分钟，SS表示秒。在给DATETIME类型的字符赋值时，可以使用**字符串类型**或者**数字类型**的数据插入，只要符合DATETIME的日志格式即可。

1. 以'YYYY-MM-DD HH:MM:SS'或者'YYYYMMDDHHMMSS'字符串格式表示的值，取值范围为'1000-01-01 00:00:00' ~ '9999-12-3 23:59:59'。

2. 以'YY-MM-DD HH:MM:SS'或者'YYMMDDHHMMSS'字符串格式表示的日期，在这里YY表示两位年值。MySQL使用一下规则解释两位年值：

   - `'00 ~ 69'`范围的年值转换为`'2000 ~ 2069'`；
   - `'70 ~ 99'`范围的年值转换为`'1970 ~ 1999'`；

3. 以YYYYMMDDHHMMSS或者YYMMDDHHMMSS数字格式表示的日期和时间。

4. 使用`NOW()`，插入当前系统日期和时间。

   > NOW()函数返回当前系统的日期和时间值，格式为'YYYY-MM-DD HH:MM:SS'。

> 提示
>
> MySQL允许“不严格”语法：任何标点符号都可以用作日期部分或时间部分之间的间隔符。例如：'98-12-31 11:30:45'、'98.12.31 11+30+45'、'98/12/31 11*30*45'和'98@12@31 11^30^45'是等价的，这些值都可以正确地插入数据库。

#### TIMESTAMP

TIMESTAMP显示的格式与DATETIME相同，显示宽度固定在19个字符，日期格式为YYYY-MM-DD HH:MM:SS，在存储时需要4字节。TIMESTAMP列的取值范围小于DATATIME的取值范围，为'1970-01-01 00:00:01' UTC ~ '2038-01-19 01:14:07' UTC。其中，UTC（Corrdinated Universal Time）为世界标准时间，因此在插入数据时，需要保证在合法的取值范围。



### 文本字符串类型

字符串类型用来存储字符串数据，除了可以存储字符串数据之外，还可以存储其他数据，比如图片和声音的二进制数据。MySQL支持两类字符型数据：文本字符串和二进制字符串。文本字符串可以进行区分或者不区分大小写的串比较，还可以进行模式匹配查找。在MySQL中，文本字符串类型是指CHAR、VARCHAR、TEXT、ENUM和SET。

*MySQL中文本字符串数据类型：*

| 类型名称   | 说明                                        | 存储需求                                                |
| ---------- | ------------------------------------------- | ------------------------------------------------------- |
| CHAR(M)    | 固定长度的非二进制字符串                    | M字节，1<=M<=255                                        |
| VARCHAR(M) | 变长非二进制字符串                          | L+1字节，在此L<=M和1<=M<=255                            |
| TINYTEXT   | 非常小的非二进制字符串                      | L+1字节，在此L<2<2^8                                    |
| TEXT       | 小的非二进制字符串                          | L+2字节，在此L<2<2^16                                   |
| MEDIUMTEXT | 中等大小的非二进制字符串                    | L+3字节，在此L<2<2^24                                   |
| LONGTEXT   | 大的非二进制字符串                          | L+4字节，在此L<2<2^32                                   |
| ENUM       | 枚举类型，只能有一个枚举字符串值            | 1或2字节，取决于枚举值的数目（最大值为65535）           |
| SET        | 一个设置，字符串对象可以有零个或多个SET成员 | 1、2、3、4或8字节，取决于集合成员的数量（最多为个成员） |

#### 1. CHAR和VARCHAR类型

CHAR(M)为固定长度字符串，在定义时指定字符串列长。当保存时在右侧填充空格，以达到指定的长度。M表示列长度，M的范围是0~255个字符。例如，CHAR(4)定义了一个固定长度的字符串列，其包含的字符个数最大为4。当检索到CHAR值时，尾部的空格将被删除。

VARCHAR(M)是长度可变的字符串，M表示最大列长度。M的范围是0~65535。VARCHAR的最大实际长度是由最长的行的大小和使用的字符集确定，而其实占用的空间为字符串的实际长度加1。例如VARCHAR(50)定义了一个最大长度为50的字符串，如果插入的字符串只有10个字符，则实际存储的字符串为10个字符和一个字符串结束字符。VARCHAR在值保存和检索时尾部的空格仍保留。

如下表格将说明CHAR和VARCHAR之间的差别：将不同字符串保存到CHAR(4)和VARCHAR(4)列。

| 插入值     | CHAR(4)  | 存储需求 | VARCHAR(4) | 存储需求 |
| ---------- | -------- | -------- | ---------- | -------- |
| `''`       | `'    '` | 4字节    | `''`       | 1字节    |
| `'ab'`     | `'ab  '` | 4字节    | `'ab'`     | 3字节    |
| `'abc'`    | `'abc '` | 4字节    | `'abc'`    | 4字节    |
| `'abcd'`   | `'abcd'` | 4字节    | `'abcd'`   | 5字节    |
| `'abcdef'` | `'abcd'` | 4字节    | `'abcd'`   | 5字节    |

对比结果可以看到，CHAR(4)定义了固定长度为4的列，不管存入的数据长度为多少，所占用的空间均为4字节；VARCHAR(4)定义的列所占的字节数为实际长度加1。



#### 2. TEXT类型

#### 3. ENUM类型

#### 4. SET类型

# MySQL权限与安全管理

## 权限表

MySQL服务通过权限表来控制用户对数据库的访问，权限表存放在MySQL数据库中，由MySQL_install_db脚本初始化。

存储账户权限信息的表主要有`user`、`db`、`host`、`tables_priv`、`columns_priv`和`procs_priv`。

### user表

user表是MySQL中最重要的一个权限表，记录允许连接到服务器的账号信息，里面的权限是全局的。

MySQL 8.0中user表有42个字段，这些字段可以分为4类，分别是用户列、权限列、安全列和资源控制列。

user表结构

| 字段名 | 数据类型 | 默认 |
| --------------------- | ------------- | ---- |
| `Host`                | `char(60)`    | `''` |
| `User`                | `char(32)`   | `''` |
| `Select_priv`         | `enum('N','Y')` | `N` |
| `Insert_priv`         | `enum('N','Y')` | `N` |
| `Update_priv`         | `enum('N','Y')` | `N` |
| `Delete_priv`         | `enum('N','Y')` | `N` |
| `Create_priv`         | `enum('N','Y')` | `N` |
| `Drop_priv`           | `enum('N','Y')` | `N` |
| `Reload_priv`         | `enum('N','Y')` | `N` |
| `Shutdown_priv`       | `enum('N','Y')` | `N` |
| `Process_priv`        | `enum('N','Y')` | `N` |
| `File_priv`           | `enum('N','Y')` | `N` |
| `Grant_priv`          | `enum('N','Y')` | `N` |
| `References_priv`     | `enum('N','Y')` | `N` |
| `Index_priv`          | `enum('N','Y')` | `N` |
| `Alter_priv`          | `enum('N','Y')` | `N` |
| `Show_db_priv`        | `enum('N','Y')` | `N` |
| `Super_priv`          | `enum('N','Y')` | `N` |
| `Create_tmp_table_priv` | `enum('N','Y')` | `N` |
| `Lock_tables_priv`    | `enum('N','Y')` | `N` |
| `Execute_priv`        | `enum('N','Y')` | `N` |
| `Repl_slave_priv`     | `enum('N','Y')` | `N` |
| `Repl_client_priv`    | `enum('N','Y')` | `N` |
| `Create_view_priv`    | `enum('N','Y')` | `N` |
| `Show_view_priv`      | `enum('N','Y')` | `N` |
| `Create_routine_priv` | `enum('N','Y')` | `N` |
| `Alter_routine_priv`  | `enum('N','Y')` | `N` |
| `Create_user_priv`    | `enum('N','Y')` | `N` |
| `Event_priv`          | `enum('N','Y')` | `N` |
| `Trigger_priv` | `enum('N','Y')` | `N` |
| `Create_tablespace_priv` | `enum('N','Y')` | `N` |
| `ssl_type` | `enum('','ANY','X509','SPECIFIED')` | `''` |
| `ssl_cipher` | `blob` |  |
| `x509_issuer` | `blob` |  |
| `x509_subject` | `blob` |  |
| `max_questions` | `int(11) unsigned` | `0` |
| `max_updates` | `int(11) unsigned` | `0` |
| `max_connections` | `int(11) unsigned` | `0` |
| `max_user_connections` | `int(11) unsigned` | `0` |
| `plugin` | `char(64)` | `mysql_native_password` |
| `authentication_string` | `text` | `NULL` |
| `password_expired` | `enum('N','Y')` | `N` |
| `password_last_changed` | `timestamp` | `NULL` |
| `password_lifetime` | `smallint(5)` | `NULL` |
| `account_locked` | `enum('N','Y')` | `N` |

#### 用户列

user表的用户列包括Host、User、authentication_string，分别表示如下：

- `Host`：主机名
- `User`：用户名
- `authentication_string`：密码

User和Host为User表的联合主键。

当用户与服务器之间建立连接时，输入的账户信息中的用户名称、主机名和密码必须完全匹配，才允许建立连接。

这3个字段的值就是创建账户时保存的账户信息。

修改密码时，实际就是修改user表的`authentication_string`字段的值。

#### 权限列

权限列的字段决定了用户的权限，描述了在全局范围内允许对数据和数据库进行的操作。

包括查询权限、修改权限等普通权限，还包括了关闭服务器、超级权限和加载用户等高级权限。普通权限用户操作数据库；高级权限用户数据库管理。

user表中对应的权限是针对所有用户数据库的。这些字段值的类型为`ENUM`，可以取得值只能为`Y`和`N`，`Y`表示该用户有对应的权限；`N`表示用户没有对应的权限。查询user表的结构可以看到，这些字段的值默认都是`N`。如果要修改权限，可以使用`GRANT`语句或`UPDATE`语句更改user表的这些字段来修改用户对应的权限。

#### 安全列

安全列只有6个字段，其中两个是ssl相关的，两个时x509相关的，另外两个是授权插件相关的。

- ssl用户加密；
- x509标准可用于标识用户；
- Plugin字段标识可以用于验证用户身份的插件，如果该字段为空，服务器使用内建授权验证机制验证用户身份。

可以通过`SHOW VARIABLES LIKE 'have_openssl;'`语句来查询服务器是否支持ssl功能。

#### 资源控制列

资源控制列的字段用来限制用户使用的资源，包含4个字段，分别为：

- `max_questions`：用户每小时允许执行的查询操作次数。
- `max_updates`：用户每小时允许执行的更新操作次数。
- `max_connections`：用户每小时允许执行的连接操作次数。
- `max_user_connections`：用户允许同时建立的连接次数。

一个小时内用户查询或者连接数量超过资源控制的限制，用户将被锁定，直到下一个小时，才可以执行的对应的操作。

可以使用`GRANT`语句更新这些字段的值。

### db表

db表示MySQL数据库中非常重要的权限表。db表中存储了用户对某个数据库的操作权限，决定用户能从哪个主机存取哪个数据库。

db表比较常用。

*db表结构如下：*

| 字段名                  | 数据类型        | 默认 |
| ----------------------- | --------------- | ---- |
| `Host`                  | char(60)        | ''   |
| `Db`                    | char(64)        | ''   |
| `User`                  | char(32)        | ''   |
| `Select_priv`           | `enum('N','Y')` | `N`  |
| `Insert_priv`           | `enum('N','Y')` | `N`  |
| `Update_priv`           | `enum('N','Y')` | `N`  |
| `Delete_priv`           | `enum('N','Y')` | `N`  |
| `Create_priv`           | `enum('N','Y')` | `N`  |
| `Drop_priv`             | `enum('N','Y')` | `N`  |
| `Grant_priv`            | `enum('N','Y')` | `N`  |
| `References_priv`       | `enum('N','Y')` | `N`  |
| `Index_priv`            | `enum('N','Y')` | `N`  |
| `Alter_priv`            | `enum('N','Y')` | `N`  |
| `Create_tmp_table_priv` | `enum('N','Y')` | `N`  |
| `Lock_tables_priv`      | `enum('N','Y')` | `N`  |
| `Create_view_priv`      | `enum('N','Y')` | `N`  |
| `Show_view_priv`        | `enum('N','Y')` | `N`  |
| `Create_routine_priv`   | `enum('N','Y')` | `N`  |
| `Alter_routine_priv`    | `enum('N','Y')` | `N`  |
| `Execute_priv`          | `enum('N','Y')` | `N`  |
| `Execute_priv`          | `enum('N','Y')` | `N`  |
| `Trigger_priv`          | `enum('N','Y')` | `N`  |

#### 用户列

db表用户列有3个字段，分别是Host、User、Db，标识从某个主机连接某个用户对某个数据库的操作权限，这个3个字段的组合构成了db表的主键。

host表不存储用户名称，用户列，只有2个字段，分别Host和Db，表示从某个主机连接的用户对某个数据库的操作权限，其主键包括Host个Db两个字段。host很少用到，一般情况下db表就可以满足权限控制需要求。

#### 权限列

db表中`Create_routine_priv`、`Alter_routine_priv`这两个字段表名用户是否有创建和修改存储过程的权限。

user表中的权限是针对所有数据库的，如果希望用户只对某个数据库有操作权限，那么需要将user表中对应的权限设置为N，然后在db表中设置对应数据库的操作权限。

### tables_priv表和columns_priv表

- tables_priv表用来对表设置操作权。
- columns_priv表用来对表的某一列设置权限。

*tables_priv表结构如下：*

| 字段名        | 数据类型                                                     | 默认值              |
| ------------- | ------------------------------------------------------------ | ------------------- |
| `Host`        | `char(60)`                                                   | ''                  |
| `Db`          | `char(64)`                                                   | ''                  |
| `User`        | `char(32)`                                                   | ''                  |
| `Table_name`  | `char(64)`                                                   | ''                  |
| `Grantor`     | `char(93)`                                                   | ''                  |
| `Timestamp`   | `timestamp`                                                  | `CURRENT_TIMESTAMP` |
| `Table_priv`  | `set('Select','Insert','Update','Delete','Create','Drop','Grant','References','Index','Alter','Create View','Show view','Trigger')` | ''                  |
| `Column_priv` | `set('Select','Insert','Update','References')`               | ''                  |

*columns_priv表结构如下：*

| 字段名        | 数据类型                                       | 默认值              |
| ------------- | ---------------------------------------------- | ------------------- |
| `Host`        | `char(60)`                                     | ''                  |
| `Db`          | `char(64)`                                     | ''                  |
| `User`        | `char(32)`                                     | ''                  |
| `Table_name`  | `char(64)`                                     | ''                  |
| `Column_name` | `char(64)`                                     | ''                  |
| `Timestamp`   | `timestamp`                                    | `CURRENT_TIMESTAMP` |
| `Column_priv` | `set('Select','Insert','Update','References')` | ''                  |

tables_priv表有8个字段，分别是Host、Db、User、Table_name、Grantor、Timestamp、Table_priv和Column_priv。

各字段说明如下：

1. Host、Db、User、Table_name4个字段分别表示主机名、数据库名、用户名和表名。
2. Grantor表示修改该记录的用户。
3. Timestamp字段表示修改该记录的时间。
4. Table_priv表示对表的操作权限，包括`Select`,`Insert`,`Update`,`Delete`,`Create`,`Drop`,`Grant`,`References`,`Index`,`Alter`,`Create View`,`Show view`,`Trigger`。
5. Column_priv字段标识对表中的列的操作权限，包括`Select`,`Insert`,`Update`,`References`。

columns_priv表只有7个字段，其中`Column_name`用来指定对那些数据列具有操作权限。

### procs_priv表

procs_priv表可以对存储过程和存储函数设置操作权限。

*procos_priv表结构如下：*

| 字段名         | 数据类型                               | 默认值            |
| -------------- | -------------------------------------- | ----------------- |
| `Host`         | char(60)                               | ''                |
| `Db`           | char(64)                               | ''                |
| `User`         | char(32)                               | ''                |
| `Routine_name` | char(64)                               | ''                |
| `Routine_type` | enum('FUNCTION','PROCEDURE')           | NOT NULL          |
| `Grantor`      | char(93)                               | ''                |
| `Proc_priv`    | set('Execute','Alter Routine','Grant') | ''                |
| `Timestamp`    | timestamp                              | CURRENT_TIMESTAMP |

procs_priv表包含8个字段，各字段说明如下：

1. Host、Db和User字段分别表示主机名，数据库名和用户名。
2. Routine_name表示存储过程或函数的名称。
3. Routine_type表示存储过程或函数的类型。Routine_type字段有两个值，分别是`FUNCTION`,`PROCEDURE`。
   1. `FUNCTION`：表示这是一个函数。
   2. `PROCEDURE`：表示这是一个存储过程。
4. Grantor是插入或修改记录的用户。
5. Proc_priv表示拥有的权限，包括`Execute`,`Alter Routine`,`Grant`3种。
6. Timestamp表示更新记录的时间。

## 账户管理



### 登录和登出MySQL服务器

可以通过MySQL --help命令查看MySQL命令的帮助信息。

MySQL命令的常用参数如下：

1. `-h`：主机名，可以使用该参数指定主机名或ip，如果不指定，默认是localhost。
2. `-u`：用户名，可以使用该参数指定用户名。
3. `-p`：密码，可以使用该参数指定登录密码。如果该参数后面有一段字符串，则该段字符串将作为用户的密码直接登录。如果后面没有内容，则登录的时候会提示输入密码。**注意：该参数后面的字符串和-p之间不能有空格**。
4. `-P`：端口号，该参数后面接MySQL服务器的端口号，默认为3306。
5. `数据库名`：可以在命令的最后指定数据库名。
6. `-e`：执行SQL语句，如果指定了该参数，将在登录后执行-e后面的命令或SQL语句并退出。

示例：使用root用户登录到本地MySQL服务器的MySQL数据库中。

```shell
$ mysql -h localhost -u root -p mysql
Enter password: *********
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 26
Server version: 5.7.20 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

执行命令时，会提示Enter password:，如果没有设置密码，可以直接按Enter键。如果设置了，密码正确就可以直接登录到服务器下面的mysql数据库中。

### 新建普通用户

#### 1. 使用CREATE USER语句创建新用户

执行CREATE USER或GRANT语句时，服务器会修改相应的用户授权表，添加或者修改用户及其权限。

CREATE USER语法格式：

```mysql
CREATE USER user_specification [,user_specification] ...
```

```mysql
user_specification: 
	user@host [
        IDENTIFIED BY [PASSWORD] 'password'
      | IDENTIFIED WITH auth_plugin [BY|AS 'auth_string']
    ]
```

- `user`：表示创建的用户的名称。
- `host`：表示允许登录的用户主机名。
- `IDENTIFIED BY`：表示用来设置用户密码。
- `[PASSWORD]`：表示使用哈希值设置密码，该参数可选。
- `'password'`：表示用户登录时使用的普通明文密码。
- `IDENTIFIED WITH`：用于为用户指定一个身份认证插件。
- `auth_plugin`：插件的名称，插件的名称可以是一个带单引号的字符串或者带双引号的字符串。
- `auth_string`：是可选的字符串参数，前面可以选择对应的关键字BY和AS。
  - 如果前面的关键字为BY，则会将该参数将传递给身份验证插件，由插件解释该参数的含义。
  - 如果前面的关键字是AS，则会将该参数原封不动的插入到表中。

CREATE USER语句会添加一个新的MySQL账户。使用CREATE USER语句，用户必须有全局的CREATE USER权限或者MySQL数据库的INSERT权限。每添加一个用户，CREATE USER语句会在MySQL.user表中添加一条新记录，但是新创建的账户没有任何权限。如果添加的账户已经存在，CREATE USER语句会返回一个错误。

示例：创建一个用户，用户名是jeffrey，密码是mypass，主机名是localhost。

```mysql
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY 'mypass';
CREATE USER 'jeffrey'@'localhost' IDENTIFIED WITH mysql_native_password BY 'mypass';
```

如果只指定了用户名部分'jeffrey'，则主机名部分默认为'%'，表示对所有的主机开放权限。

```mysql
CREATE USER 'jeffrey'
```

user_specification告诉MySQL服务器当用户登录时怎么验证用户的登录授权，如果指定用户登录不需要密码，则可以省略IDENTIFIED BY部分：

```mysql
CREATE USER 'jeffrey'@'localhost'
```

> MySQL的某些版本中会引入授权表的结构变化，添加新的特权或功能。每当更新MySQL到一个新的版本时，应该更新授权表，以确保它们有最新的结构，确认可以使用任何新功能。

#### 2. 直接操作MySQL.user表

使用CREATE USER创建新用户时，实际上都是在user表中添加一条新的记录，因此，可以使用INSERT语句向user表中直接插入一条记录来创建一个新的用户。使用INSERT语句，必须拥有对mysql.user表的INSERT权限。

使用INSERT语句创建新用户的基本语法格式如下：

```mysql
INSERT INTO mysql.user (`Host`,`User`,`authentication_string`,ssl_cipher,x509_issuer,x509_subject) VALUES ('host','username',PASSWORD('password'),'','','');
```

`ssl_cipher`、`x509_issuer`、`x509_subject`三个字段是NOT NULL，必须设值，所以这里设置为空。

### 删除普通用户

#### 1. 使用DROP USER语句删除用户

DROP USER语法格式：

```mysql
DROP USER user [, user];
```

DROP USER语句用于删除一个或多个MySQL账户。要使用DROP USER，必须拥有MySQL数据库全局DROP    USER权限或者DELETE权限。使用与GRANT或REVOKE相同的格式为每个账户命名。例如，"'jeffrey'@'localhost'"账户名称的用户和主机部分与用户表记录的User和Host列值相对应。

使用DROP USER，可以删除一个账户和其权限，操作如下：

```mysql
DROP USER 'user'@'localhost';
DROP USER;
```

上述两个语句的作用如下：

1. 第1条语句可以删除user在本地登录权限。
2. 第2条语句可以删除来自所有授权表的账户权限记录。

> **提示**
>
> DROP USER不能自动关闭任何打开的用户对话。而且，如果用户有打开的对话，此时取消用户，命令则不会生效，直到用户对话被关闭后才能生效。一对话被关闭，用户也被取消，此用户再次视图登录时将会失败。

#### 2. 使用DELETE语句删除用户

DELETE删除用户语法格式：

```mysql
DELETE FROM mysql.user WHERE `User`='username' AND `Host`='host';
```

### root用户修改自己的密码

通过UPDATE语句修改mysql.user表的authentication_string字段，修改用户密码：

```mysql
UPDATE mysql.user SET authentication_string=PASSWORD('mypass') WHERE `User`='jeffrey' AND `Host`='localhost';

FLUSH PRIVILEGES;
```

`PASSWORD()`函数用来加密用户密码。执行UPDATE语句之后，需要执行`FLUSH PRIVILEGES`重新加载用户权限。

### root用户修改普通用户的密码

#### 1. 使用SET语句修改普通用户的密码

语法格式如下：

```mysql
SET PASSWORD FOR 'user'[@'host']='password';
```

示例：

```mysql
SET PASSWORD FOR 'jeffrey'@'localhost'='mypass';
```

#### 2. 使用UPDATE语句修改普通用户的密码

```mysql
UPDATE mysql.user SET authentication_string=PASSWORD('mypass') WHERE `User`='jeffrey' AND `Host`='localhost';

FLUSH PRIVILEGES;
```

`PASSWORD()`函数用来加密用户密码。执行UPDATE语句之后，需要执行`FLUSH PRIVILEGES`重新加载用户权限。







```mysql
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '597646251';
```











