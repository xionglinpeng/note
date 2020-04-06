# Docker实战







## Seagull

Githup：<https://github.com/tobegit3hub/seagull>

**介绍**

Seagull是友好的Web UI，用于管理和监控docker。

- 易于在docker容器中安装和卸载。
- 一键启动/停止/删除容器和镜像。
- 超快（<10ms）— 搜索和过滤。
- 支持i18n国际化，包括英语、中文、德语和法语。

**安装**

```shell
$ docker run -d -p 10086:10086 -v /var/run/docker.sock:/var/run/docker.sock tobegit3hub/seagull
```

或者运行`docker-compose up -d`。

**多主机**

Seagull支持监控多个服务器。

```shell
$ docker -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -api-enable-cors=true -d
```

## UI-for-docker

Githup：<https://github.com/kevana/ui-for-docker>

**介绍**

Docker的web接口，以前称为DockerUI。这个repo未得到维护。

> UI-for-docker已经被废弃，最新维护的是[portainer](https://github.com/portainer/portainer)。

**快速开始**

1. 运行：

   ```shell
   $ docker run -d -p 9000:9000 --privileged -v /var/run/docker.sock:/var/run/docker.sock uifd/ui-for-docker
   ```

2. 打开浏览器：`http://<dockerd-host-ip>:9000`。

**以指定的套接字连接到Docker守护进程**

默认情况下，UI-For-Docker使用`/var/run/docker.sock`来连接到Docker守护进程。因此，您需要使用`-v /var/run/docker.sock:/var/run/docker.sock`将unix套接字绑定到容器中。

可以使用`-H`标签来更改此套接字：

```shell
# 连接到tcp套接字
$ docker run -d -p 9000:9000 --privileged uifd/ui-for-docker -H tcp://127.0.0.1:2375
```

有关更多信息，请查看Githup。



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

   

### Nginx





**搭建文件下载服务器**

*配置文件*

```nginx
worker_processes  1;
events {
    worker_connections  1024;
}
http {
	server {
		listen       80;
		server_name  localhost;
		charset utf-8;
		location / {
			root /opt/files/resources;
			autoindex on;
			autoindex_exact_size off;
			autoindex_format html;
			autoindex_localtime on;
	   }
	}
}
```

*docker command line*

```shell
docker run \
	-d \
	-p 80:80 \
	--name nginx-alpine \
	-v /mnt/sda1/var/lib/home/download:/opt/files/resources \
	-v /mnt/sda1/var/lib/home/nginx/config/nginx.conf:/etc/nginx/nginx.conf \
	--restart=always \
	nginx:1.17.9-alpine
```

*docker-compose.yml*

```yaml
version: "3"
services:
  nginx:
    image: nginx:1.17.9-alpine
    container_name: "nginx-alpine"
    ports:
      - "80:80"
    volumes:
      - "/mnt/sda1/var/lib/home/download:/opt/files/resources"
      - "/mnt/sda1/var/lib/home/nginx/config/nginx.conf:/etc/nginx/nginx.conf"
    restart: always
```

需要被下载的文件放置在`/mnt/sda1/var/lib/home/download`目录下，访问http://ip-host即可看到下载管理界面。





### Harbor

Harbor是一个开源的容器镜像注册表，它是通过基于角色的访问控制来保护镜像，扫描镜像中的漏洞并将镜像签名受信任的。作为CNCF孵化项目，Harbor提供合规性，性能和互操作性，以帮助您跨Kubernetes和Docker等云原生计算平台持续，安全地管理镜像。

> 简单的说就是自定义的镜像注册中心，只不过它的功能更加强大。

Official：https://goharbor.io/

Githup：https://github.com/goharbor/harbor

Demo：https://demo.goharbor.io

### Weavescope



### Portainer





Official：https://www.portainer.io/

Githup：https://github.com/portainer



## Universal Control Plane

Universal Control Plane简称UCP，它跟portainer一样，也是一个Docker管理的UI工具。不过不同的是UCP是由Docker官方提供的Docker企业版特性，也就是说它是收费的。

UCP提供了一个月的免费使用许可，可以用于学习使用。

Official：https://docs.docker.com/ee/ucp/

使用许可下载：https://hub.docker.com/bundles/docker-datacenter

## K8S

### Wayne

Wayne 是一个通用的、基于 Web 的 **Kubernetes 多集群管理平台**。通过可视化 Kubernetes 对象模板编辑的方式，降低业务接入成本， 拥有完整的权限管理系统，适应多租户场景，是一款适合企业级集群使用的**发布平台**。

Wayne已大规模服务于360搜索，承载了内部绝大部分业务，稳定管理了近千个业务，上万个容器，运行了两年多时间，经受住了生产的考验。

> Wayne就是360提供的一个对Kubernetes进行管理的UI工具。

Offiicial（貌似不可用）：https://360yun.org/

Githup：https://github.com/Qihoo360/wayne