# 在Windows上使用noinstall ZIP存档安装MySQL

从noinstall软件包安装的用户可以使用本节中的说明来手动安装mysql。从zip档案包安装Mysql的过程如下：

1. 将存档解压缩到所需的安装目录

2. 创建一个选项文件（my.ini）

3. 选择一个mysql服务类型

4. 启动MySql服务器

5. 确保默认用户账户

这个过程在下面的部分中描述。

## 提取装档案

要手动安装mysql，请执行以下操作：

1. 如果您从以前的版本升级，请参阅 ，然后在开始升级过程。

2. 确保您已具有管理员权限的用户身份登录。

3. 选择一个安装位置。传统上，MySql服务器安装在`C:mysql`。如果您没有安装在`C:mysql`，则必须在启动期间或在选项文件中指定安装目录的路径，请参见[2.3.5.2、创建一个选项文件](创建一个选项文件.md)。

   >  注意
   >
   > ​            MySql安装程序安装在C:Program FilesMySQL

4. 使用首选的文件压缩工具将安装zip文件解压缩到选定的安装位置，某些工具可能会将存档提取到您选择的安装位置的文件夹中。如果发生这种情况，您可以将子文件夹的内容移动到选定的安装位置。

## 创建一个选项文件

如果您在运行服务器时需要指定启动项，则可以在命令行上指明它们，或将它们放在选项文件当中。对于每次服务器启动时使用的选项，您可能会发现使用选项文件来指定您的MySQL配置是最方便的。在以下情况下尤其如此：

- 安装或数据目录位置与默认位置`（C:Program FilesMySQLMySQL Server 5.6和 C:Program FilesMySQLMySQL Server 5.6data）`不同。

- 您需要调整服务器设置，例如内存，缓存或InnoDB配置信息。

当MySql服务器在Windows上启动时，它会在多个位置（例如Windows目录C:）和MySql安装目录查找选项文件（有关位置的完整列表，请参见 第4.2.6节“使用选项文件”）。Windows目录通常被命名为类似C:WINDOWS。您可以WINDIR使用以下命令从环境变量的值确定其确切位置 ：

```shell
C:> echo %WINDIR%
```

MySQL首先在my.ini文件中查找每个位置的选项，然后在my.cnf文件中查找。但是，为了避免混淆，最好只使用一个文件。如果您的电脑使用的引导装载程序c:不是引导驱动器，则唯一的选择是使用该my.ini文件。无论使用哪个选项文件，它都必须是存文本文件。

**注意**

> 当MySQL安装程序安装MySQL服务器时，它将my.ini文件在默认位置创建。而从MySQL Server 5.5.27起，运行MySQL Installer的用户被授予对这个新的完全权限my.ini。
>
> 换句话说，确保MySQL服务器用户有权读取my.ini文件。

你也可以使用MySQL发行版中包含的示例选项文件；请参见。

可以使用任何文件编辑器（如记事本）创建和修改选项文件。例如，如果安装了MySQL `E:mysql`并且数据目录处于`E:mydatadata`，则可以创建包含一个`[mysqld]`部分的选项文件，以指定`basedir`和`datadir`选项的值：

```
[mysqld]
# set basedir to your installation path
basedir=E:/mysql
# set datadir to the location of your data directory
datadir=E:/mydata/data
```

Microsoft Windows路径名是在选项文件中使用（向前）斜线而不是反斜线来指定的。如果你使用反斜杠，把它们加倍：

```
[mysqld]
# set basedir to your installation path
basedir=E:mysql
# set datadir to the location of your data directory
datadir=E:mydatadata
```

选项文件值中使用反斜线的规则在第4.2.6节“使用选项文件”中给出。

数据目录位于AppData运行MySQL的用户的 目录中。

如果您想在其他位置使用数据目录，则应将data目录的全部内容复制 到新位置。例如，如果您想要E:mydata用作数据目录，则必须执行两项操作：

1. 将整个data目录及其所有内容从默认位置（例如 `C:Program FilesMySQLMySQL Server 5.6data`）移到`E:mydata`。

2. `--datadir`每次启动服务器时 使用一个选项来指定新的数据目录位置。

## 选择一个MySQL服务器类型

下表显示了MySQL 5.6中可用的Windows服务器。

| 二进制       | 描述                                          |
| ------------ | --------------------------------------------- |
| mysqld       | 使用命名管道支持优化二进制文件                |
| mysqld-debug | 像mysql一样，但编译完全调试和自动内存分配检查 |

所有上述二进制文件都针对现代英特尔处理器进行了优化，但应该可以在任何Intel i386或更高级别的处理器上使用。

发行版中的每个服务器都支持相同的一组存储引擎，使用[show engines](https://dev.mysql.com/doc/refman/5.6/en/show-engines.html) 语句显示给定服务器支持的引擎。

所有Windows MySQL 5.6服务器都支持数据库目录的符号链接。

MySQL支持所有Windows平台上的TCP / IP。如果使用该--enable-named-pipe选项启动服务器，Windows上的MySQL服务器也支持命名管道。有必要明确地使用这个选项，因为有些用户在使用命名管道时遇到了关闭MySQL服务器的问题。默认情况下使用TCP / IP而不考虑平台，因为在许多Windows配置中，命名管道比TCP / IP慢。

## 首次启动服务器

本节给出了启动MySQL服务器的一般概述。以下各节提供了有关从命令行启动MySQL服务器或作为Windows服务的更多特定信息。

这里的信息主要适用于你使用该noinstall版本安装MySQL ，或者如果你想手动配置和测试MySQL而不是使用GUI工具。

**注意**

>MySQL服务器在使用MySQL安装程序后会自动启动， MySQL通知程序可以随时用来启动/停止/重新启动。

这些部分中的示例假定MySQL安装在默认位置C:Program FilesMySQLMySQL Server 5.6。如果您将MySQL安装在不同的位置，请调整示例中显示的路径名称。

客户有两个选择。他们可以使用TCP / IP，或者如果服务器支持命名管道连接，他们可以使用命名管道。

如果使用该--shared-memory选项启动服务器，则MySQL for Windows也支持共享内存连接 。客户端可以使用该--protocol=MEMORY选项通过共享内存进行连接 。

有关要运行哪个服务器二进制文件的信息，请参见 第2.3.5.3节“选择MySQL服务器类型”。

最好通过控制台窗口（或“ DOS窗口 ”）中的命令提示符进行测试。通过这种方式，您可以让服务器在易于查看的窗口中显示状态消息。如果您的配置有问题，这些消息使您更容易识别和修复任何问题。

要启动服务器，请输入以下命令：

~~~
C:> "C:Program FilesMySQLMySQL Server 5.6binmysqld" --console
~~~

对于包含InnoDB支持的服务器，您应该在启动时看到类似于以下消息的消息（路径名称和大小可能不同）：

~~~
InnoDB: The first specified datafile c:ibdataibdata1 did not exist:
InnoDB: a new database to be created!
InnoDB: Setting file c:ibdataibdata1 size to 209715200
InnoDB: Database physically writes the file full: wait...
InnoDB: Log file c:iblogsib_logfile0 did not exist: new to be created
InnoDB: Setting log file c:iblogsib_logfile0 size to 31457280
InnoDB: Log file c:iblogsib_logfile1 did not exist: new to be created
InnoDB: Setting log file c:iblogsib_logfile1 size to 31457280
InnoDB: Log file c:iblogsib_logfile2 did not exist: new to be created
InnoDB: Setting log file c:iblogsib_logfile2 size to 31457280
InnoDB: Doublewrite buffer not found: creating new
InnoDB: Doublewrite buffer created
InnoDB: creating foreign key constraint system tables
InnoDB: foreign key constraint system tables created
011024 10:58:25  InnoDB: Started
~~~

当服务器完成其启动序列时，您应该看到类似这样的内容，这表示服务器已准备好为客户端连接提供服务：

~~~
mysqld: ready for connections
Version: '5.6.40'  socket: ''  port: 3306
~~~

服务器继续向控制台写入任何进一步的诊断输出。您可以打开一个新的控制台窗口来运行客户端程序。

如果省略该--console选项，则服务器将诊断输出写入数据目录中的错误日志（C:Program FilesMySQLMySQL Server 5.6data默认情况下）。错误日志是具有.err扩展名的文件，可以使用该--log-error 选项进行设置。

**注意**

>MySQL授权表中列出的帐户最初没有密码。启动服务器之后，应使用第2.10.4节“确保初始MySQL帐户的安全”中的说明为它们设置密码 。

## 从Windows命令行启动MySQL

MySQL服务器可以从命令行手动启动。这可以在任何版本的Windows上完成。

**注意**

>MySQL通告程序也可以用来启动/停止/重新启动MySQL服务器。

要从命令行启动mysqld服务器，您应该启动一个控制台窗口（或“ DOS窗口 ”）并输入以下命令：

~~~
C:> "C:Program FilesMySQLMySQL Server 5.6binmysqld"
~~~

mysqld 的路径可能会有所不同，具体取决于系统上MySQL的安装位置。

您可以通过执行以下命令来停止MySQL服务器：

~~~
C:> "C:Program FilesMySQLMySQL Server 5.6binmysqladmin" -u root shutdown
~~~

**注意**

>如果MySQL root用户帐户有密码，则需要 使用该选项调用mysqladmin，-p并在出现提示时提供密码。

该命令调用MySQL管理实用程序 mysqladmin连接到服务器，并告诉它关闭。该命令以MySQL root用户身份连接，MySQL 用户是MySQL授权系统中的默认管理帐户。

**注意**

>MySQL授权系统中的用户完全独立于Microsoft Windows下的任何登录用户。

如果mysqld没有启动，请检查错误日志，看看服务器是否在那里写了任何消息来指出问题的原因。默认情况下，错误日志位于`C:Program FilesMySQLMySQL Server 5.6data`目录中。它是带有后缀的文件.err，或者可以通过传递--log-error 选项来指定。或者，您可以尝试使用该`--console`选项启动服务器 ; 在这种情况下，服务器可能会在屏幕上显示一些有用的信息，以帮助解决问题。

最后一个选项是使用 和 选项启动mysqld。在这种情况下， mysqld写入一个日志文件，该文件 应包含mysqld无法启动的原因。请参见 第24.5.3节“DBUG包”。` --standalone--debugC:mysqld.trace`

使用`mysqld --verbose --help`显示mysqld支持的所有选项。

## 自定义MySQL工具的PATH

为了更容易调用MySQL程序，可以将MySQL bin目录的路径名添加到Windows系统PATH环境变量中（`简单的说就是设置环境变量`）：

* 在Windows桌面上，右键单击我的电脑图标，然后选择 `属性`。

* 接下来从出现的“` 系统属性`”菜单中选择“ `高级`”选项卡，然后单击“` 环境变量`” 按钮。 

* 在“ `系统变量`”下，选择“` 路径`”，然后单击“ `编辑`”按钮。在编辑系统变量对话框应该会出现。

* 将光标放在出现在标记为“ `变量值`”的空白处的文本的末尾。（使用 `End键`确保你的光标位于这个空间的文本的最后）。然后输入你的`MySQL bin`目录的完整路径名 （例如 `C:Program FilesMySQLMySQL Server 5.6bin`）

**注意**

>这个路径中必须有一个分号来区分这个字段中的任何值。

忽略这个对话，然后依次单击OK，直到所有打开的对话都被解除。这个新的 PATH值现在可以用于你打开的任何新的命令shell，允许你通过在DOS提示符下从系统的任何目录输入它的名字来调用任何MySQL可执行程序，而不必提供路径。这包括服务器， mysql客户端以及所有MySQL命令行实用程序，如`mysqladmin`和` mysqldump`。

如果您在同一台计算机上运行多个MySQL服务器， 则不应将MySQL bin目录添加到Windows PATH。

## 将MySQL作为Windows服务器启动

## 测试MySQL安装

您可以通过执行以下任一命令来测试MySQL服务器是否正在工作：

~~~
C:> "C:Program FilesMySQLMySQL Server 5.6binmysqlshow"
C:> "C:Program FilesMySQLMySQL Server 5.6binmysqlshow" -u root mysql
C:> "C:Program FilesMySQLMySQL Server 5.6binmysqladmin" version status proc
C:> "C:Program FilesMySQLMySQL Server 5.6binmysql" test
~~~

如果mysqld响应来自客户端程序的TCP / IP连接速度很慢，那么DNS可能有问题。在这种情况下， 使用该 选项启动`mysqld--skip-name-resolve`并仅使用MySQL授权表localhost的Host列中的IP地址。（请确保存在指定IP地址的帐户，否则您可能无法连接。）

通过指定`--pipe`或 `--protocol=PIPE`选项，或指定.（句点）作为主机名，可以强制MySQL客户端使用命名管道连接而不是TCP / IP 。`--socket`如果您不想使用默认管道名称，请使用该选项指定管道的名称。

如果已经设置了密码的root 帐户，删除匿名帐户，或创建新的用户帐户，然后连接到MySQL服务器，你必须使用适当的-u，并-p选择与前面显示的命令。请参见 第4.2.2节“连接到MySQL服务器”。

有关mysqlshow的更多信息，请参见 第4.5.6节“ mysqlshow - 显示数据库，表和列信息”。

## 总结

## noinstall安装

本文档安装测试版本为5.7.20，可能不同的版本有所差异。

```
C:Usersxlp>mysql --version
mysql  Ver 14.14 Distrib 5.7.20, for Win64 (x86_64)
```

### 1、下载

从[mysql官方网站](http://www.mysql.com/)下载noinstall软件包安装（即一个zip压缩包）

`DOWNLOADS` > `Windows` > `MySQL Installer` > `MySQL Server` > 

```
Windows (x86, 64-bit), ZIP Archive` > `No thanks, just start my download.
```

然后开始下载

### 2、解压

将下载完成之后的压缩包解压到需要安装的目录。

### 3、环境变量

将其`bin/`目录配置为环境变量。

### 4、创建数据目录

创建一个数据目录，此数据目录用于存放mysql数据库文件。

例如：`D:programMysqldata`

### 5、添加my.ini

mysql配置文件，默认没有my.ini文件，手动创建一个。然后在my.ini文件中添加如下配置：

```
[mysqld]
# set basedir to your installation path
basedir=D:/program/Mysql
# set datadir to the location of your data directory
datadir=D:/program/Mysql/data
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

### 6、初始化

执行命令`mysqld.exe --initialize`初始化mysql，这一步操作是用于生成mysql系统数据库及相关日志文件等，生成在第4步创建的数据库目录中。

执行完成后会在数据目录（`D:programMysqldata`）中生成一个 主机名`.err`文件，里面有一个记录root用户密码的记录，密码为随机生成。

例如：

```
2018-04-20T14:23:27.901130Z 1 [Note] A temporary password is generated for root@localhost: WCv:55euFsDa
```

### 7、启动

执行命令`mysqld.exe --console`启动数据库。

### 8、修改密码

新打开一个cmd窗口，用在`.err`文件中生成的随机密码登录修改密码。

```
#登录
mysql -uroot -p
#修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

### 9、安装服务

再次新打开一个cmd窗口（以管理员身份打开）。

安装服务：

```
D:developerMysqlbin>mysqld.exe --install
Service successfully installed.
```

如果之前有安装，需要先卸载服务：

```
C:WINDOWSsystem32>sc delete MySQL
[SC] DeleteService 成功
```

注意：

>如果没有管理员身份打开cmd窗口，将不能安装服务。

### 10、启动

```
D:developerMysqlbin>net start mysql
MySQL 服务正在启动 .
MySQL 服务已经启动成功。
```

注意：

>此处也要是在以管理员身份打开的cmd窗口上执行，否则将会出错：

```
D:developerMysqlbin>net start mysql
发生系统错误 5。
拒绝访问。
```

## 实践经验

项目中本来使用的是mysql5.6进行开发，切换到5.7之后，突然发现原来的一些sql运行都报错，错误编码1055，错误信息和sql_mode中的“only_full_group_by“有关，到网上看了原因，说是mysql5.7中only_full_group_by这个模式是默认开启的 

解决办法大致有两种： 

一：在sql查询语句中不需要group by的字段上使用any_value()函数 

当然，这种对于已经开发了不少功能的项目不太合适，毕竟要把原来的sql都给修改一遍

二：修改my.cnf（windows下是my.ini）配置文件，删掉only_full_group_by这一项 

我们项目的mysql安装在ubuntu上面，找到这个文件打开一看，里面并没有sql_mode这一配置项，想删都没得删。 

当然，还有别的办法，打开mysql命令行，执行命令

select @@sql_mode

1

这样就可以查出sql_mode的值，复制这个值，在my.cnf中添加配置项（把查询到的值删掉only_full_group_by这个选项，其他的都复制过去）：

sql_mode=STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

1

如果 [mysqld] 这行被注释掉的话记得要打开注释。然后重重启mysql服务

注：使用命令

set sql_mode=STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

1

这样可以修改一个会话中的配置项，在其他会话中是不生效的。





























