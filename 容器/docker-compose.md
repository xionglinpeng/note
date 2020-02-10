# Docker-compose

官方文档：https://docs.docker.com/compose/

Githup：https://github.com/docker/compose

二进制包：https://github.com/docker/compose/releases

## 安装docker-compose

2. 二进制包

   docker官方发布的docker-compose二进制包可以在githup的https://github.com/docker/compose/releases页面找到，截止（2020-1-25）当前最新版本1.25.3。将二进制包下载到执行路径下，并添加可执行权限即可。

   ```shell
   # 下载相应版本的二进制包
   $ curl -L https://github.com/docker/compose/releases/download/1.25.3/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
   # 设置可执行权限
   $ chmod +x /usr/local/bin/docker-compose
   ```

   执行`docker-compose version`测试是否OK。

   ```shell
   $ docker-compose version
   docker-compose version 1.25.3, build d4d1b42b
   docker-py version: 4.1.0
   CPython version: 3.7.5
   OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
   ```

   

3. 镜像安装

   ```shell
   [root@localhost ~]# curl -L https://github.com/docker/compose/releases/download/1.25.3/run.sh > /usr/local/bin/docker-compose
     % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
   100   596    0   596    0     0     77      0 --:--:--  0:00:07 --:--:--   188
   100  1789  100  1789    0     0    120      0  0:00:14  0:00:14 --:--:--   494
   
   $ chmod +x docker-compose
   
   
   
   ```

   