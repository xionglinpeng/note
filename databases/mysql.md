# MySQL


## error
1. mysql8.0链接报错-caching_sha2_password
  error description: 安装的mysql版本为mysql8.0，安装完成之后使用SQLyog连接，始终连接不上，报错`caching_sha_password`。

  查看mysql配置文件，发现身份认证方式发生了改变

  将`default-authentication-plugin=mysql_native_password`前面的#去掉

  重启mysql服务，这样再创建用户，默认会采用`mysql_native_password`

  对于已经存在的用户，还是原来的认证方式

  登录mysql，修改已创建用户的认证方式
  ```mysql
  alter user root@localhost identified WITH mysql_native_password by 'password';
  ```
  刷新数据库权限，再去连接数据库就可以了
  ```mysql
  flush privileges;
  ```
引用：https://jingyan.baidu.com/article/6dad5075284acfa123e36e35.html
