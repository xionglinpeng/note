RabbitMQ

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














