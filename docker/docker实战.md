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

Docker hub：https://hub.docker.com/_/mysql

特性说明：

**mysql容器相关目录**

- `/var/lib/mysql`：数据目录，必须挂载此目录，不然容器重建，数据就会丢失。
- `/etc/mysql/conf.d`：自定义配置的配置目录，在此目录下放置以`.cnf`结尾的文件，在容器启动时将会被加载。
- `/var/lib/mysql-files`：容器启动会报`Failed to access directory for --secure-file-priv`错误，然后启动失败，应该是没权限，没这个问题的就不用挂载这个目录。

MySQL的默认配置位于`/etc/mysql/my.cnf`，包括其他目录`/etc/mysql/conf.d`或`/etc/mysql/mysql.conf.d`。

`/etc/mysql/conf.d`目录中的配置文件和默认配置文件`/etc/mysql/my.cnf`将会同时起作用，但前者优先。

**环境变量**

- ##### `MYSQL_ROOT_PASSWORD`：数据库密码。

- ##### `MYSQL_DATABASE`：该变量是可选的，允许您指定镜像启动时创建的数据库的名称。如果提供了用户/密码(见下面)，那么该用户将被授予超级用户

- ##### `MYSQL_USER`，`MYSQL_PASSWORD`：这些变量是可选的，用于创建新用户和设置该用户的密码。这个用户将被授予`MYSQL_DATABASE`变量指定的数据库的超级用户权限(见上面)。这两个变量都是创建用户所必需的。请注意，不需要使用此机制来创建根超级用户，该用户在默认情况下使用`MYSQL_ROOT_PASSWORD`变量指定的密码创建。

- ##### `MYSQL_ALLOW_EMPTY_PASSWORD`：可选值。允许ROOT用户密码为空，*非空值*，例如yes。注意：不建议这样做。

- ##### `MYSQL_RANDOM_ROOT_PASSWORD`：可选值。随机生成ROOT用户密码，*非空值*，例如yes。密码将被打印到stdout (GENERATED ROOT PASSWORD: .....)。

- ##### `MYSQL_ONETIME_PASSWORD`：强制用户更改密码。一旦init完成，将root(不是MYSQL_USER!中指定的用户)用户设置为过期，强制在第一次登录时更改密码。任何*非空值*都将激活该设置。注意:此功能仅在MySQL 5.6+上支持。在MySQL 5.5上使用这个选项将在初始化期间抛出一个适当的错误。

- ##### `MYSQL_INITDB_SKIP_TZINFO`：默认情况下，entrypoint脚本自动加载`CONVERT_TZ()`函数所需的时区数据。如果不需要，任何*非空值*都将禁用时区加载。

**没有cnf配置**

许多配置选项可作为命令行参数传递给mysqld。这样可以灵活的定义容器，而不需要cnf配置文件。例如使用UTF-8（utf8mb4）改变所有表的默认编码和排序规则，只需要运行一下代码：

```shell
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

查看可用选项的完整列表

```shell
$ docker exec -it [container_name || container_id] mysql --verbose --help
```

**connect to mysql**

```shell
$ docker exec -it [container_name || container_id] mysql -h [host] -u[user] -p[password]
```

**docker command line**

```shell
$ docker run \
	-p 3306:3306 \
	-e MYSQL_ROOT_PASSWORD=123456 \
	--restart always \
	--privileged \
	--net [network-name] \
	--memory 300M \
	--name db-mysql \
	-v /opt/mysql/datadir:/var/lib/mysql \
	-v /opt/mysql/conf:/etc/mysql/conf.d \
	-v /opt/mysql/mysql-files:/var/lib/mysql-files \
	-d \
	mysql:debian
```

**docker-compose.yaml**

```yaml
version: '3'
services:
  mysql:
    image: mysql:8.0.21
    container_name: db-mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: a6ce8a15fe164d51b30923c6914ef9cc
    restart: always
    privileged: true
    volumes:
      # 数据目录
      - /opt/mysql/datadir:/var/lib/mysql
      # 配置目录
      - /opt/mysql/conf:/etc/mysql/conf.d
      # secure-file-priv 参数值
      - /opt/mysql/mysql-files:/var/lib/mysql-files
    networks:
      - [network-name]
networks:
  [network-name]:
    external: true
```

**my.cnf**

```mysql
[mysqld]
sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
```

mysql更多系统变量：http://dev.mysql.com/doc/mysql/en/server-system-variables.html

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

   ```mysql
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

   ```mysql
   mysql> flush privileges;
   ```

## NoSQL数据库

### Redis

Docker Hub : https://hub.docker.com/_/redis?tab=description

特性说明：

- redis容器数据目录位于`/data`。

- 默认情况下没有配置文件，需要自定义配置文件，并映射到容器的`/etc/redis/redis.conf`。

- 启动Redis容器开启持久化：追加容器启动命令`redis-server --appendonly yes`。
- 启动Redis容器时指定配置文件`redis-server /etc/redis/redis.conf --appendonly yes`。

*docker command line*

```shell
$ docker run \
	--name no-sql-redis \
	--privileged \
	--restart always \
	-p 6379:6379 \
	-v /opt/redis/data:/data \
	-v /opt/redis/conf/redis.conf:/etc/redis/redis.conf \
	--net [network] \
	--network-alias no-sql-redis \
	-d \
	redis:6.0.7-alpine3.12 \
	redis-server /etc/redis/redis.conf --appendonly yes
```

*docker-compose.yaml*

```yaml
version: '3'
services:
  no-sql: 
    image: redis:6.0.7-alpine3.12
    container_name: no-sql-redis
    ports:
      - "6379:6379"
    restart: always
    privileged: true
    volumes:
      - /opt/redis/data:/data
      - /opt/redis/conf/redis.conf:/etc/redis/redis.conf
    command: ["redis-server","/etc/redis/redis.conf","--appendonly","yes"]
    networks:
      [network]:
        aliases:
          - no-sql-redis
networks:
  [network]:
    external: true
```

*connect to redis*

```shell
$ docker exec -it [container_name || container_id] redis-cli
$ docker exec -it [container_name || container_id] redis-cli -h [host]
$ docker exec -it [container_name || container_id] redis-cli -h [host] -p 6379
$ docker exec -it [container_name || container_id] redis-cli -h [host] -p 6379 -a [password]
```

## Web服务器

### Nginx

Docker Hub : https://hub.docker.com/_/nginx

特性说明：

- nginx镜像配置文件位于`/etc/nginx/nginx.conf`。
- 默认数据目录位于`/usr/share/nginx/html`。
- nginx常常需要将一些不同的资源放置在自定义的目录中，用以区分，这是自定义数据卷挂载即可。
- nginx镜像还包含了其他很多高级特性，但一般情况下都是用不到的，具体参考[Docker Hub](https://hub.docker.com/_/nginx)。

*docker-compose.yaml*

```yaml
version: '3'
services:
  web:
    image: nginx:1.19.2-alpine
    container_name: web-nginx
    ports:
      - "80:80"
    restart: always
    privileged: true
    volumes:
      - /usr/share/nginx/html:/usr/share/nginx/html:ro
      - /usr/share/nginx/config/nginx.conf:/etc/nginx/nginx.conf:ro
      - [volume-name]:/usr/share/nginx/resource:ro
    networks:
      - [network-name]
networks:
  [network-name]:
    external: true
volumes:
  [volume-name]:
    external: true
```

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



## docker-registry-web

docker-registry-web是私有的Docker Registry v2的web UI。

特性：

- 在docker registry v2中浏览repositories, tags和images。
- Optional token based authentication provider with role-based permissions。
- Docker registry通知记录和审核。

Githup：<https://github.com/mkuchin/docker-registry-web>

Docker Hup：<https://hub.docker.com/r/hyper/docker-registry-web/>

**Docker拉取命令**

```shell
$ docker pull hyper/docker-registry-web
```

**如何运行**

**Qucik start（使用环境变量进行配置，无身份认证）**

```shell
# 首先运行registry:2容器
$ docker run -d -p 5000:5000 --name registry-srv registry:2
# 再运行docker-registry-web
$ docker run -it \
	-p 8080:8080 \
	--name registry-web \
	--link registry-srv \
	-e REGISTRY_URL=http://registry-srv:5000/v2 \
	-e REGISTRY_NAME=localhost:5000 \
	hyper/docker-registry-web 
```

> 不要使用`registry`作为registry容器的名称，否则会破坏`REGISTRY_NAME`环境变量。

使用基本身份认证和自签名证书连接到docker registry。

```shell
$ docker run -it -p 8080:8080 --name registry-web --link registry-srv \
           -e REGISTRY_URL=https://registry-srv:5000/v2 \
           -e REGISTRY_TRUST_ANY_SSL=true \
           -e REGISTRY_BASIC_AUTH="YWRtaW46Y2hhbmdlbWU=" \
           -e REGISTRY_NAME=localhost:5000 \
           hyper/docker-registry-web
```

**无身份认证，使用配置文件**

1. 创建配置文件`config.yml`

   ```yaml
   registry:
     # Docker registry url
     url: http://registry-srv:5000/v2
     name: localhost:5000
     # 运行镜像删除，应该是false
     readonly: false
     auth:
     	# 禁用身份认证
       enabled: false
   ```

   > 此配置中的所有属性都可以被环境变量覆盖。例如`registry.auth.enabled`将被环境变量`REGISTRY_AUTH_ENABLED`覆盖。

2. 运行docker

   ```shell
   $ docker run -d -p 5000:5000 --name registry-srv registry:2
   $ docker run -it \
   	-p 8080:8080 \
   	--name registry-web \
   	--link registry-srv \
   	-v $(pwd)/config.yml:/conf/config.yml:ro \
   	hyper/docker-registry-web
   ```

3. `http://localhost:8080`访问Web UI。

**启用身份认证**





### Harbor

Harbor是一个开源的容器镜像注册表，它是通过基于角色的访问控制来保护镜像，扫描镜像中的漏洞并将镜像签名受信任的。作为CNCF孵化项目，Harbor提供合规性，性能和互操作性，以帮助您跨Kubernetes和Docker等云原生计算平台持续，安全地管理镜像。

> 简单的说就是自定义的镜像注册中心，只不过它的功能更加强大。

Official：https://goharbor.io/

Githup：https://github.com/goharbor/harbor

Demo：https://demo.goharbor.io

### Weavescope





## Portainer

Portainer是一个用于构建，管理和维护Docker环境的UI工具，Portainer可以为软件开发人员和IT操作人员提供直观的界面。Portainer提供了对Docker环境的详细概述，并可以管理容器，镜像，网络和数据卷。Portainer易于部署——仅需要一个Docker命令即可以在任何地方运行Portainer。
Portainer是免费的，它有一些扩展功能，例如Portainer Registry Manager，只不过这些扩展功能是需要收费的。

此外，Portainer官方还提供了Windows容器主机的部署方式，但是我们一般都是使用的Linux容器主机，所以这里不做介绍，如想要了解，请参考官方[安装指南](https://www.portainer.io/installation/)。

Official：https://www.portainer.io/

Official Document：<https://portainer.readthedocs.io/en/stable/>

Githup：https://github.com/portainer

### Docker安装Portainer

Portainer由两个元素组成——Portainer服务和Portainer代理。这两个元素都作为Docker容器在Docker中运行。由于Docker的特性，存在多种部署方案，下面将介绍最常用的部署方案（如果以下部署方案没有您所需要的，请参考portainer.readthedocs.io以获取其他选项）。

请注意，使用Docker Swarm模式时，推荐使用Portainer代理。

**单机Linux Docker主机部署**

使用以下Docker命令部署Portainer服务器；请注意，在单机Docker主机上不需要代理。

```shell
$ docker volume create protainer_data
$ docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

部署完成之后访问9000端口即可。

>  注意：`-v /var/run/docker.sock:/var/run/docker.sock`选项只能在Linux环境使用。

**Docker Swarm集群部署**

Swarm集群部署需要部署Portainer服务和Portainer Agent代理，Portainer官方提供了相应的stack.yml文件，直接下载部署即可，如下：

```shell
$ curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml
$ docker stack deploy --compose-file=portainer-agent-stack.yml portainer
```

部署完成之后访问9000端口即可。

> 注意：Portainer Agent将作为全局服务部署在集群的每个节点上。

**只部署Portainer Agent**

在远程Linux Swarm集群部署Protainer Agent集群服务，在远程集群的管理节点上运行以下命令：

```shell
$ docker service create --name portainer_agent --network portainer_agent_network --publish mode=host,target=9001,published=9001 -e AGENT_CLUSTER_ADDR=tasks.portainer_agent --mode global --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock --mount type=bind,src=//var/lib/docker/volumes,dst=/var/lib/docker/volumes –-mount type=bind,src=/,dst=/host portainer/agent
```







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