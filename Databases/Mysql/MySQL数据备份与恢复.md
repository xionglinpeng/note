



# 数据备份与恢复

数据备份时数据库管理员非常重要的工作之一，系统以外崩溃或者硬件的损坏都可能导致数据库丢失，因此MySQL管理员应该定期地备份数据库，使得在以外情况发生时，竟可能减少损失。

## 一、数据备份

MySQLdump是MySQL提供的一个非常有用的数据库备份工具。

MySQLdump名称执行时，可以将数据库备份成一个文本文件，该文件中实际包含了多个CREATE和INSERT语句 ，使用这些语句可以重新创建表和插入数据。

### 1. 使用MySQLdump命令备份

语法格式：

```shell
$ mysqldump -h <host> -u <user> -p<password> -P <port> <dbname> [tbname tbname ...] > /path/filename.sql
```

- `<host>`：登录用户的主机名称。
- `<user>`：用户名称。
- `<password>`：登录密码。
- `<port>`：服务端口号。
- `<dbname>`：需要备份的数据库名称。
- `tbname`：为<dbname>数据库中需要备份的数据库表，可以指定多个。
- `>`：右箭头符号“>”告诉MySQLdump将备份的数据表的定义和数据写入备份文件。
- `/path`：备份文件所在路径。
- `filename.sql`：备份文件的名称。

#### ① 使用MySQLdump备份单个数据库中的所有表

语法格式：

```shell
$ mysqldump -h <host> -u <user> -p<password> -P <port> [dbbame] > /path/filename.sql
```

> 提示
>
> 这里要保证/path文件夹存在，否则将提示错误信息：系统找不到指定的路径。

示例：

```shell
$ mysqldump -h 192.168.56.10 -uroot -p597646251 -P 31306 ApolloConfigDB > C:\Users\xlp\Desktop\ApolloConfigDB.sql
```

#### ② 使用MySQLdumo备份数据库中的某个表

语法格式：

```shell
$ mysqldump -h <host> -u <user> -p<password> -P <port> <dbname> [tbname tbname ...] > /path/filename.sql
```

tbname表示数据库中的表名，多个表名之间用空格隔开。

> 备份表和备份数据库中所有表的语句的不同之处在于，要在数据库名称dbname之后指定需要备份的表名称。

示例：

```shell
$ mysqldump -h 192.168.56.10 -uroot -p597646251 -P 31306 ApolloConfigDB ServerConfig > C:\Users\xlp\Desktop\ServerConfig.sql
```

#### ③ 使用MySQLdump备份多个数据库

语法格式：

```shell
$ mysqldump -h <host> -u <user> -p<password> -P <port> --databases [dbname dbname ...] > /path/filename.sql
```

`--databases`参数之后必须只是指定一个数据库名称，多个数据库名称使用空格隔开。

示例：

```shell
$ mysqldump -h 192.168.56.10 -uroot -p597646251 -P 31306 --databases ApolloConfigDB ApolloPortalDB > C:\Users\xlp\Desktop\ApolloDB.sql
```

使用`--all-databases`参数备份系统中的所有数据，不需要知道数据库名称。

语法格式：

```shell
$ mysqldump -h <host> -u <user> -p<password> -P <port> --all-databases > path/filename.sql
```

示例：

```shell
$ mysqldump -uroot -p597646251 --default-character-set=utf8 --all-databases > C:\Users\xlp\Desktop\localhost1.sql
```

#### ④ 备份文件

备份文件其内容大致如下：

```mysql
-- MySQL dump 10.13  Distrib 8.0.20, for Win64 (x86_64)
--
-- Host: 192.168.56.10    Database: ApolloConfigDB
-- ------------------------------------------------------
-- Server version	8.0.20

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!50503 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `AccessKey`
--

......

/*!40000 ALTER TABLE `ServerConfig` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2020-06-20 13:00:09
```

备份文件包含了以下信息：

- 文件开头表明了备份文件使用MySQLdump工具的版本号。

- 备份账户的名称和主机信息

- 备份的数据库的名称

- MySQL服务器的版本号

- 备份文件接下来的部分试一下SET语句，这些语句将一些系统变量赋值给用户定义变量，以确保被回复的数据库的系统变量和原来备份时的变量相同。

  例如：

  ```mysql
  /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
  ```

  该语句将当前系统变量`character_set_client`的值赋给用户定义变量`@character_set_client`。其他变量与此类似。

- 备份文件的最后几行MySQL使用SET语句回复服务器系统变量原来的值。

  例如：

  ```mysql
  /*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
  ```

  该语句将用户定义的变量`@character_set_client`中保存的值赋给实际的系统变量`character_set_client`。

- 备份文件中以“--”字符开头的行为注释语句；

- 以“`/*`”开头，“`*/`”结尾的语句为可执行的MySQL注释，这些语句可被MySQL执行，但在其他数据库管理系统中将被作为注释忽略，以提高数据库的可移植性。

- 备份文件开始的一些语句以数字开头，代表的是MySQL版本号，这些语句只有在指定的MySQL版本或者比该版本高的情况下才能执行。例如40101，表明这些语句只有在MySQL版本号为4.01.01或者更高的条件下才可以被执行。

### 2. 直接复制整个数据库目录

## 二、数据恢复

管理人员操作的失误，计算机故障以及其他以外情况，都会导致数据的丢失和破坏。当数据丢失或以外破坏时，可以通过恢复已经备份的数据尽量减少数据丢失和破坏造成的损失。

### 1. 使用MySQL命令恢复

语法格式：

```shell
$ mysql -h <host> -u user -p<password> -P <port> [dbname] < /path/fileName.sql
```

- `<host>`：登录用户的主机名称。
- `<user>`：用户名称。
- `<password>`：登录密码。
- `<port>`：服务端口号。
- `[dbname]`：需要导入的数据库名称。

- `<`：左箭头符号“<”声明数据导入的SQL文件。
- `/path`：SQL文件所在路径。
- `fileName.sql`：SQL文件的名称。

> 如果fileName.sql文件为MySQLdump工具创建的包含创建数据库语句的文件，执行的时候不需要指定数据库库名。

示例：

```shell
$ mysql -h localhost -u root -p597646251 -P 3306 < C:\Users\xlp\Desktop\localhost.sql
```

如果已经登录MySQL服务器，还可以使用`source`命令导入sql文件。

source语法格式：

```shell
mysql> source filename
```

示例：

```shell
mysql> source C:\Users\xlp\Desktop\localhost.sql
```

> 提示：
>
> 执行source命令前，必须使用use语句选择数据。不然，恢复过程中会出现“ERROR 1046 (3D000)：No database selected”错误。

### 2. 直接复制到数据库目录

### 3. MySQLhotcopy快速恢复



## 四、表的导入导出

### 1. 使用SELECT...INTO OUTFILE导出文本文件

`SELECT...INTO OUTFILE`语法格式：

```mysql
SELECT columnlist FROM table WHERE condition INTO OUTFILE 'filename' [OPTIONS];

    -- OPTIONS选项
    FIELDS 
    	TERMINATED BY 'value'
        [OPTIONALLY] ENCLOSED BY 'value'
        ESCAPED BY 'value'
    LINES 
    	STARTING BY 'value'
        TERMINATED BY 'value'
```

语法格式说明：

- `SELECT columnlist FROM table WHERE condition`：一个查询语句，查询结果返回满足指定条件的一条或多条记录；
- `INTO OUTFILE`：其作用是把前面SELECT语句查询出来的结果导出到名称为`'filename'`的外部文件中；
- `[OPTIONS]`：可选参数项；
  1. `FIELDS TERMINATED BY 'value'`：设置字段之间的分隔字符，可以为单个或多个字符，默认情况下为制表符‘`\t`’。
  2. `FIELDS [OPTIONALLY] ENCLOSED BY 'value'`：设置字段的包围字符，只能为单个字符。若使用了`OPTIONALLY`，则只有`CHAR`和`VARCHAR`等字符数据字段被包括。
  3. `FIELDS ESCAPED BY 'value'`：控制如何写入或读取特殊字符，只能为单个字符，即设置转义字符，默认为“`\`”。
  4. `LINES STARTING BY 'value'`：设置每行数据开头的字符，可以为单个或多个字符，默认情况下不使用任何字符。
  5. `LINES TERMINATED BY 'value'`：设置每行数据结尾的字符，可以为单个或多个字符，默认为‘`\n`’。

注意事项：

- `'filename'`不能是一个已经存在的文件。
- `'filename'`路径分隔符必须是“`/`”，不能是“`\`”。
- 必须拥有文件写入权限。

- `FIELDS`和`LINES`两个子句都是自选的，但是如果两个都被指定了，FIELDS必须位于LINES之前。

- `SELECT...INTO OUTFILE`语句可以非常快速地把一个表转储到服务器上。如果想要在服务器主机之外的部分客户主机上转储文件，不能使用SELECT...INTO OUTFILE，在这种情况下，应该在客户主机上使用比如“`mysql -e "SELECT ..." > file_name`”的命令来生成文件。

示例：

```mysql
SELECT * FROM USER INTO OUTFILE 'D:/Temp/user.txt' 
FIELDS
	TERMINATED BY ',' 
	OPTIONALLY ENCLOSED BY '\"'
	ESCAPED BY '\\'
LINES 
	STARTING BY '<'
	TERMINATED BY '>\n';
```

执行报错：

```txt
The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

这是因为MySQL默认对导出的目录有权限限制。

查询目录配置：

```mysql
mysql> SHOW GLOBAL VARIABLES LIKE '%secure%';
+--------------------------+----------+
| Variable_name            | Value    |
+--------------------------+----------+
| require_secure_transport | OFF      |
| secure_file_priv         | NULL     |
+--------------------------+----------+
2 rows in set, 1 warning (0.00 sec)
```

`secure_file_priv`值为NULL。至mysql 5.6.34版本以后`secure_file_priv`的值默认为NULL。并且无法用sql语句对其进行修改，只能够通过以下方式修改：

- **windows**

  修改`my.ini`文件，在`[mysqld]`下添加条目：

  ```shell
  secure-file-priv="D:/Temp"
  ```

  保存，重启mysql。

- **Linux**

  在`/etc/my.cnf`的`[mysqld]`下面添加选项：

  ```shell
  local-infile=0
  ```

  保存，重启mysql。

更新配置之后，重启mysql。再次查询目录配置：

```mysql
mysql> SHOW GLOBAL VARIABLES LIKE '%secure%';
+--------------------------+----------+
| Variable_name            | Value    |
+--------------------------+----------+
| require_secure_transport | OFF      |
| secure_file_priv         | D:\Temp\ |
+--------------------------+----------+
2 rows in set, 1 warning (0.00 sec)
```

### 2. 使用MySQLdump命令导出文本文件

MySQLdump命令不仅可以将数据导出为包含CREATE、INSERT的sql文件，也可以导出为纯文本文件。

使用MySQLdump命令导出文本文件或同时导出sql文件和文本文件。

MySQLdump导出文本文件语法格式：

```shell
$ mysqldump -h <host> -u <user> -p<password> -P <port> <dbname> [taname taname taname ...] -T <path> [options]
	-- OPTIONS 选项
	
```

参数说明：

- `<host>`：mysql服务器主机。
- `<user>`：mysql服务器用户名。
- `<password>`：mysql服务器密码。
- `<port>`：mysql服务器端口。
- `<dbname>`：数据库名称。
- `[taname]`：表名称，可选，可以指定多个，如果不指定，将导出数据库dbname中的所有的表。
- `<path>`：导出文件的路径。
- `[options]`：可选参数选项，这些选项需要结合-T选项使用。
  - `--fields-terminated-by=value`：设置字段之间的分隔符，可以为单个或多个字符，默认情况下为制表符“`\t`”。
  - `--fields-enclosed-by=value`：设置字段的包围字符。
  - `--fields-optionally-enclosed-by=value`：设置字段的包围字符，只能为单个字符，只能包括`CHAR`和`VARCHAR`等字符数据字段，
  - `--fields-escaped-by=value`：控制如何写入或读取特殊字符，只能为单个字符，即设置转义字符，默认值为反斜线“`\`”。
  - `--lines-terminated-by=value`：设置每行数据结尾的字符，可以为单个或多个字符，默认值为“`\n`”。

> 提示：
>
> 与SELECT...INTO OUTFILE语句中的OPTIONS各个参数设置不同，这里OPTIONS各个选项等号后面的value值不要用引号括起来。

示例：

```shell
$ mysqldump -h localhost -u root -p597646251 -P 3306 test_mybatis user -T D:\Temp --fields-terminated-by=, --fields-enclosed-by=\" --fields-escaped-by=$ --lines-terminated-by=<end>\n
```

### 3. 使用MySQL命令导出文本文件

如果MySQL服务器是单独的机器，用户是在一个client上进行操作，用户要把数据结果导入到client机器上，可以使用`mysql -e`语句。

语法格式：

```shell
$ mysql -h <host> -u <user> -p<password> -P <pory> --execute "select语句" dbname > filename.txt
```

- `--execute`：表示执行该选项后面的语句并退出，后面的语句必须用双引号括起来。
- `dbname`：要导出的数据库名称。

导出的文件中不同列之间使用制表符分隔，第一行包含了各个字段的名称。

示例：

```shell
$ mysql -h localhost -u root -p597646251 -P 3306 --execute "select * from user where id in (1,2)" test_mybatis > D:\Temp\user.txt
```

**多行显示**

使用MySQL命令还可以指定查询结果的显示格式，如果某行记录字段很多，可能一行不能完全显示，可以使用`--vertical`参数，将每条记录分为多行显示。示例：

```shell
$ mysql -h localhost -u root -p597646251 -P 3306 --vertical --execute "select * from user where id in (1,2)" test_mybatis > D:\Temp\user.txt
```

执行之后文件内容显示如下：

```text
*************************** 1. row ***************************
         id: 1
       name: User1
   password: test1
      phone: 18700001111
  nick_name: User1
create_time: 2020-01-29 22:29:08
*************************** 2. row ***************************
         id: 2
       name: User2
   password: test2
      phone: 18700001112
  nick_name: User2
create_time: 2020-02-29 22:29:08
```

**导出为html**

MySQL可以将查询结果导出到html文件中，使用`--html`选项即可。

示例：

```shell
$ mysql -h localhost -u root -p597646251 -P 3306 --html --execute "select * from user where id in (1,2)" test_mybatis > D:\Temp\user.html
```

**导出为xml**

MySQL可以将查询结果导出到xml文件中，使用`--xml`选项即可。

示例：

```shell
$ mysql -h localhost -u root -p597646251 -P 3306 --xml --execute "select * from user where id in (1,2)" test_mybatis > D:\Temp\user.xml
```

### 4. 使用LOAD DATA INFILE方式导入文本文件

MySQL允许将数据导出到外部文件，也可以从外部文件导入数据。

MySQL提供了一些导入数据的工具，包括LOAD DATA语句、source命令和mysql命令。

LOAD DATA INFILE语句用于高速地从一个文本文件中读取行，并装入一个表中。

文件名称必须为文字字符串。

LOAD DATA语法格式：

```mysql
LOAD DATA INFILE 'filename.txt' INTO TABLE tbname [OPTIONS] [IGNORE number LINES]
	-- OPTIONS选项
	FIELDS 
		TERMINATED BY 'value'
        [OPTIONALLY] ENCLOSED BY 'value'
        ESCAPED BY 'value'
	LINES 
		STARTING BY 'value'
		TERMINATED BY 'value'
```

选项说明：

- 关键字`INFILE`后面的`filename`文件为导入数据的来源。
- `tbname`表示待导入的数据表的名称。
- `[OPTIONS]`为可选参数选项。
  - `FIELDS TERMINATED BY 'value'`：设置字段之间的分隔字符，可以为单个或多个字符，默认情况下为制表符‘`\t`’。
  - `FIELDS [OPTIONALLY] ENCLOSED BY 'value'`：设置字段的包围字符，只能为单个字符。若使用了`OPTIONALLY`，则只有`CHAR`和`VARCHAR`等字符数据字段被包括。
  - `FIELDS ESCAPED BY 'value'`：控制如何写入或读取特殊字符，只能为单个字符，即设置转义字符，默认为“`\`”。
  - `LINES STARTING BY 'value'`：设置每行数据开头的字符，可以为单个或多个字符，默认情况下不使用任何字符。
  - `LINES TERMINATED BY 'value'`：设置每行数据结尾的字符，可以为单个或多个字符，默认为‘`\n`’。
- `[IGNORE number LINES]`选项表示忽略文件开始处的行数，`number`表示忽略的行数。

执行LOAD DATA语句需要FILE权限。

示例：

```mysql
LOAD DATA INFILE 'D:/Temp/user.txt' INTO TABLE USER 
FIELDS 
	TERMINATED BY ','
	OPTIONALLY ENCLOSED BY '\"'
	ESCAPED BY '\\'
LINES
	STARTING BY '<'
	TERMINATED BY '>\n'
IGNORE 2 LINES;
```

### 5. 使用MySQLimport命令导入文本文件

使用MysqlImport可以导入文本文件，并且不需要登录MySQL客户端。

MySQLimport命令提供许多与`LOAD DATA INFILE`语句相同的功能，大多数选项直接对应`LOAD DATA INFILE`子句。

使用MySQLimport语句需要指定所需的选项、导入的数据库名称以及导入的数据文件的路径和名称。

MySQLimport命令语法格式：

```shell
$ mysqlimport -h <host> -u <user> -p<password> -P <port> dbname /path/filename.txt [OPTIONS]
  # -- OPTIONS
  --fields-terminated-by=value
  --fields-enclosed-by=value
  --fields-optionally-enclosed-by=value
  --fields-escaped-by=value
  --lines-terminated-by=value
  --ignore-lines=value
  -d
  --verbose
```

选项说明：

- `--fields-terminated-by=value`：设置字段之间的分隔字符，可以为单个或多个字符，默认情况下为制表符“`\t`”。
- `--fields-enclosed-by=value`：设置字段的包围字符。
- `--fields-[optionally]-enclosed-by=value`：设置字段的包围字符，只能为单个字符，包括`CHAR`和`VARCHAR`等字符数据字段。
- `--fields-escaped-by=value`：控制如何写入或读取特殊字符，只能为单个字符，即设置转义字符，默认为“`\`”。
- `--lines-terminated-by=value`：设置每行数据结尾的字符，可以为单个或多个字符，默认为‘`\n`’。
- `--ignore-lines=value`：忽视数据文本的前n行。
- `-d --delete`：导入文本文件前清空表。
- `--verbose`：冗长模式。打印出程序操作的详细信息。

与LOAD DATA INFILE不同的是，其选项值不需要使用单引号括起来，但是该转义的还是需要转义。

示例：

```shell
$ mysqlimport -h localhost -u root -p597646251 -P3306 test_mybatis D:/Temp/user.txt \
--fields-terminated-by=, \
--fields-optionally-enclosed-by=\" \
--fields-escaped-by=\ \
--lines-terminated-by=\?\n \
--ignore-lines=1 \
-d  \
--verbose
mysqlimport: [Warning] Using a password on the command line interface can be insecure.
Connecting to localhost
Selecting database test_mybatis
Deleting the old data from table user
Loading data from SERVER file: D:/Temp/user.txt into user
test_mybatis.user: Records: 3  Deleted: 0  Skipped: 0  Warnings: 0
Deleting the old data from table test_mybatis
mysqlimport: Error: 1146, Table 'test_mybatis.test_mybatis' doesn't exist, when using table: test_mybatis
```

MySQLimport其他选项：

- `-c, --columns=name`：采用逗号分隔的列名作为其值。列名的顺序指示如何匹配数据文件列和表列。
- `-C, --compress`：压缩在客户端和服务器之间发生的所有信息（如果二者均支持压缩）
- `-f, --force`：忽视错误。例如，某个文本文件的表不存在，继续处理其他文件。不使用`--force`选项，如果表不存在，则MySQLimport退出。
- `--ignore-lines=n`：忽视数据文件的前n行。
- `-L, --local`：从本地客户端读入输入文件。
- `-l, --lock-tables`：处理文本文件前锁定所有表，以便写入。这样可以确保所有表在服务器上保持同步。
- `-r, --replace`：`-r, --replace`和`-i, --ignore`选项控制复制唯一键值已有记录的输入记录的处理，如果指定`-r, --replace`，新行替换有相同的唯一键的已有行；如果指定`-i, --ignore`，复制已有的唯一键值的输入行被跳过；如果不指定这两个选项，当发现一个复制键值时会出现一个错误，并且忽视文本文件的剩余部分。
- `-i, --ignore`：参见`-r, --replace`选项的描述。
- `--protocol=[TCP|SOCKET|PIPE|MEMORY]`：使用的连接协议。
- `-h, --host=name`：将数据导入给定主机上的MySQL服务器。默认主机是`localhost`。
- `-u, --user=name`：当连接服务器时MySQL使用的用户名。
- `-p, --password[=name]`：当连接服务器时使用的密码。如果使用短选项形式（`-p`），选项和密码之间不能有空格。如果在命令行中`-p`或者`--password选项后面没有密码值，则提示输入一个密码。
- `-P, --port=#`：用于连接接的TCP/IP端口号。
- `-s, --silent`：沉默模式。只有出现错误时才输出信息。
- `-V, --version`：显示版本信息并退出。