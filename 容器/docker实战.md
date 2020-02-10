# Docker实战





## 数据库应用



### MySQL

使用官方mysql镜像构建

```shell
$ docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=597646251 --name mysql  mysql:latest
```



**连接报错**

```
ERROR 2059 (HY000): Authentication plugin 'caching_sha2_password' cannot be loaded: ÕÒ²»µ½Ö¸¶¨µÄÄ£¿é¡£
```

这是因为在mysql 8之前的版本的加密规则是`mysql_native_password`，而在mysql 8之后加密规则是`caching_sha2_password`，所以需要把密码重新使用`mysql_native_password`加密即可。

查看mysql版本：

```shell
mysql> SHOW VARIABLES WHERE Variable_name = 'version';
+---------------+--------+
| Variable_name | Value  |
+---------------+--------+
| version       | 8.0.19 |
+---------------+--------+
```

使用`mysql_native_password`加密

1. 进入mysql容器内部

   ```shell
   $ docker exec -it mysql /bin/bash
   ```

2. 登录mysql

   ```shell
   $ mysql -u root -p
   ```

3. 查看mysql用户秘钥信息

   ```shell
   mysql> select host,user,plugin,authentication_string from mysql.user;
   +-----------+------------------+-----------------------+------------------------------------------------------------------------+
   | host      | user             | plugin                | authentication_string                                                  |
   +-----------+------------------+-----------------------+------------------------------------------------------------------------+
   | %         | root             | caching_sha2_password | $A$005$Y8f\:.M;7q	!ie~(/OO6XOT/ELEvPc0WPI/lF/ToLxOQsFL29CvI6yj2Aw6 |
   | localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
   | localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
   | localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
   | localhost | root             | caching_sha2_password | $A$005$NE	oTX}}rB!64JMHMk9Ewm68PYbdFP2B6L9ofH3GyV3dhkKlzKkVUZfqTYV6 |
   +-----------+------------------+-----------------------+------------------------------------------------------------------------+
   ```

   可以看到user为root的账户对应的plugin为`caching_sha2_password`。

4. 修改root用户密码

   ```shell
   mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '597646251';
   ```

5. 再次查询，对比结果

   ```shell
   mysql> select host,user,plugin,authentication_string from mysql.user;
   +-----------+------------------+-----------------------+------------------------------------------------------------------------+
   | host      | user             | plugin                | authentication_string                                                  |
   +-----------+------------------+-----------------------+------------------------------------------------------------------------+
   | %         | root             | mysql_native_password | *86854AAB8F0060A30AB9346F0360D335F5E53B2B                              |
   | localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
   | localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
   | localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
   | localhost | root             | caching_sha2_password | $A$005$NE	oTX}}rB!64JMHMk9Ewm68PYbdFP2B6L9ofH3GyV3dhkKlzKkVUZfqTYV6 |
   +-----------+------------------+-----------------------+------------------------------------------------------------------------+
   ```

   可以看到user为root的账户对应的plugin已经变为了`mysql_native_password`。

6. 最后刷新授权（这一步不是必须的）

   ```shell
   mysql> flush privileges;
   ```

   







