位运算符详解：https://jingyan.baidu.com/article/1612d5008ff5b7e20f1eee4c.html
java精确除法运算（BigDecimal）：https://blog.csdn.net/qq_39164396/article/details/80992577
tar.xz文件如何解压：https://blog.csdn.net/u013439115/article/details/77935602
CentOS安装新版RabbitMQ解决Erlang ：https://www.jianshu.com/p/f54dc259a9ed
在linux下安装配置rabbitMQ详细教程：https://blog.csdn.net/qq_22075041/article/details/78855708



RabbitMQ

## Erlang安装

Erlang中文教程网站：http://www.erlang-cn.com/



http://www.erlang.org/downloads

```shell
$ wget http://erlang.org/download/otp_src_21.2.tar.gz
$ tar -zxvf otp_src_21.2.tar.gz
$ cd otp_src_21.2
$ ./configure --prefix=/opt/erlang/

```

错误信息

1. 如果在`./configure`的时候报错：

   ```
   configure: error: No curses library functions found
   configure: error: /opt/erlang/otp_src_21.2/erts/configure failed for erts
   ```

   这是因为缺少ncurses-devel库，按照ncurses-devel库即可

   ```shell
   $ yum install -y ncurses-devel
   ```

如果想要通过源码安装方式解决同样问题请往下看

下载地址：<http://ftp.gnu.org/gnu/ncurses/>

我选择的是最新版本：<http://ftp.gnu.org/gnu/ncurses/ncurses-6.0.tar.gz>

```bash
[root@bogon liuzhen]# tar -zxvf ncurses-6.0.tar.gz
[root@bogon liuzhen]# cd ncurses-6.0
[root@bogon liuzhen]# ./configure --with-shared --without-debug --without-ada --enable-overwrite  
[root@bogon liuzhen]#  make
[root@bogon liuzhen]#  make install
```

```
*********************************************************************
**********************  APPLICATIONS DISABLED  **********************
*********************************************************************

jinterface     : Java compiler disabled by user


*********************************************************************
**********************  APPLICATIONS DISABLED  **********************
*********************************************************************

odbc           : ODBC library - link check failed

*********************************************************************
*********************************************************************
**********************  APPLICATIONS INFORMATION  *******************
*********************************************************************

wx             : wxWidgets not found, wx will NOT be usable

*********************************************************************
*********************************************************************
**********************  DOCUMENTATION INFORMATION  ******************
*********************************************************************

documentation  : 
                 xsltproc is missing.
                 fop is missing.
                 The documentation can not be built.

*********************************************************************
```

其中APPLICATIONS DISABLED 标示是必须要安装的，另外两个项目可以忽略
jinterface : Java compiler disabled by user
如果不用java编译可以忽略 在编译的时候 加入--without-javac

```shell
[root@localhost otp_src_21.2]# make && make install
```







## RabbitMQ安装

```shell
[root@localhost rabbitmq]# wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.10/rabbitmq-server-3.7.10-1.el7.noarch.rpm
[root@localhost rabbitmq]# yum install rabbitmq-server-3.7.10-1.el7.noarch.rpm
...
错误：软件包：rabbitmq-server-3.7.10-1.el7.noarch (/rabbitmq-server-3.7.10-1.el7.noarch)
          需要：erlang >= 19.3
 您可以尝试添加 --skip-broken 选项来解决该问题
 您可以尝试执行：rpm -Va --nofiles --nodigest
```





```shell
[root@localhost rabbitmq]# wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.10/rabbitmq-server-generic-unix-3.7.10.tar.xz
[root@localhost rabbitmq]# xz -d rabbitmq-server-generic-unix-3.7.10.tar.xz
[root@localhost rabbitmq]# tar -xvf rabbitmq-server-generic-unix-3.7.10.tar
```



```shell
[root@localhost sbin]# ./rabbitmq-server 

  ##  ##
  ##  ##      RabbitMQ 3.7.10. Copyright (C) 2007-2018 Pivotal Software, Inc.
  ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
  ######  ##
  ##########  Logs: /opt/rabbitmq/rabbitmq_server-3.7.10/var/log/rabbitmq/rabbit@localhost.log
                    /opt/rabbitmq/rabbitmq_server-3.7.10/var/log/rabbitmq/rabbit@localhost_upgrade.log

              Starting broker...
 completed with 0 plugins.
```

### 启动web管理界面

```shell
./rabbitmq-plugins enable rabbitmq-management
[root@localhost sbin]# ./rabbitmq-plugins enable rabbitmq_management
Enabling plugins on node rabbit@localhost:
rabbitmq-management
{:plugins_not_found, [:"rabbitmq-management"]}


[root@localhost sbin]# ./rabbitmq-plugins enable rabbitmq_management
Enabling plugins on node rabbit@localhost:
rabbitmq_management
The following plugins have been configured:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@localhost...
The following plugins have been enabled:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch

started 3 plugins.
```

启动

```shell
[root@localhost sbin]# ./rabbitmq-server -detached
```

停止

```shell
[root@localhost sbin]# ./rabbitmqctl shutdown
```

guest这个默认的用户只能通过http://localhost:15672 来登录，不能使用IP地址登录，也就是不能远程访问，这对于服务器上没有安装桌面的情况是无法管理维护的。要解决这个问题增加用户。

添加用户

```shell
[root@localhost sbin]# ./rabbitmqctl add_user admin admin
```

添加角色

```shell
[root@localhost sbin]# ./rabbitmqctl set_user_tags admin administrator
```

设置权限

```shell
[root@localhost sbin]# ./rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```

查看用户

```shell
[root@localhost sbin]# ./rabbitmqctl list_users
```

浏览器访问

http://192.168.1.110:15672


