# Docker-compose

官方文档：https://docs.docker.com/compose/

Githup：https://github.com/docker/compose

二进制包：https://github.com/docker/compose/releases

## Docker-compose install

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
   

## Command line

### up

### down

### -d

### scale

## docker-compose.yml



### 



### services

#### build?

##### context

指向Dockerfile文件所在目录的路径，或者git存储库的URL。

当`context`指向的路径是相对路径时，它将被解释为相对于Compose文件的位置。此目录也是发送到Docker守护进程的构建上下文。

Compose使用生成的名称构建并标记它，然后使用该映像。

```yaml
build:
  context: ./dir
```

##### dockerfile

另外的Dockerfile。

Compose使用另外的Dockerfile文件构建（必须指定构建路径—即`context`）。

```yaml
build:
  context: .
  dockerfile: Dockerfile-alternate
```



#### command



#### container_name

指定自定义容器的名称，而不是默认生成的名称。

```yaml
container_name: my-web-container
```

因为docker容器名称必须唯一，如果指定了自定义容器名称，则不能启动多容器服务。

> 注意：当使用（version 3）Compose文件以集群模式部署时，将会忽略此选项。
>
> 启动多容器服务使用docker-compose的[scale](https://docs.docker.com/compose/reference/scale/)选项，例如：
>
> ```shell
> docker-compose scale web=2 worker=3
> docker-compose up --scale web=2 worker=3
> ```
>
> 表示启动两个web容器服务，三个worker容器服务。

#### depends_on

表示服务之间的依赖关系，服务依赖关系将导致一下行为：

- `docker-compose up`按依赖顺序启动服务。在下面的示例中，`db`和`redis`先于`web`启动。
- `docker-compose stop`按依赖顺序停止。在下面的示例中，`web`先于`db`和`redis`停止。
- `docker-compose up SERVICE`自动包含`SERVICE`'s依赖的服务。在下面的示例中，`docker-compose up web`还创建和启动了`db`和`redis`。

```yaml
version: "3.7"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

> **使用`depends_on`要注意以下几点：**
>
> - `depends_on`不会在启动web之前等待db和redis“就绪”——直到它们已经启动完成。如果您需要等待服务就绪，请参阅[控制](https://docs.docker.com/compose/startup-order/)启动顺序以获得关于此问题的更多信息以及解决此问题的策略。
> - 版本3不在支持`depends_on`的`condition`格式。
> - 当使用（version 3）Compose文件以集群模式部署时，将会忽略此选项。

#### entrypoint?

#### environment?

添加环境变量，可以使用数组或字典。任何布尔值（true，false，yes，no）都需要使用引号括起来，以确保YML解析器不会将它们转换为True和False。

```yaml
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:
```

```yaml
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

> 注意：如果您的服务指定了一个[build](#build)选项，那么在构建期间，`environment`定义的变量不会自动可见。使用`build`的[args](#args)子选项来定义构建时的环境变量。

#### expose？

指定容器公开的端口，这些端口只能被链接的容器访问。

```yaml
expose:
 - "3000"
 - "8000"
```

#### external_links?

链接到在docker-compose.yml之外的容器，特别是对于公共和共享服务的容器。

```yaml
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```

> 注意：
>
> - 如果使用的是版本2以上的格式，那么链接的外部容器必须与链接到它的服务处于同一个相同的网络。[links](https://docs.docker.com/compose/compose-file/compose-file-v2/#links)是一个遗留选项。建议使用[networks](https://docs.docker.com/compose/compose-file/#networks)。
>
> - 当使用（version 3）Compose文件以集群模式部署时，将会忽略此选项。

#### extra_hosts

添加主机名映射，等价于docker客户端`--add-host`选项。

```yaml
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```

将会在此服务容器内的`/etc/hosts`文件中创建一个`ip地址 主机名`的条目。例如：

```shell
162.242.195.82  somehost
50.31.209.229   otherhost
```

#### image？

指定启动容器的镜像，可以是repository/tag，会在部分镜像ID。

```yaml
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd
```

如果指定的镜像不存在，则将会从镜像仓库中拉取。



#### ports？？

暴露端口。

> 注意：端口映射与`network_mode:host`不兼容。

**SHORT SYNTAX**

ports配置要指定两个端口（HOST:CONTAINER），要么只指定容器端口（选择临时主机端口）

> 注意：当映射HOST:CONTAINER格式的端口时，当使用小于60的容器端口时，可能出现错误的结果，因为yaml将格式为xx:yy的数字解析为base-60的值，出于这个原因，建议将端口映射显示的指定为字符串。

```yaml
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
```

**LONG SYNTAX**

长格式语法可以配置短格式不能表示的其他字段。

- `target`：容器内的端口。
- `published`：公开暴露的端口。
- `protocol`：端口的协议（`tcp`or`udp`）。
- `mode`：`host`

```yaml
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```

> 注意：长格式语法v3.2以上新的语法。



#### links

#### network_mode

网络模式。使用与docker客户端`--network`参数相同的值，加上特殊形式的`service:[service name]`。

```yaml
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

> 注意：
>
> - 当使用（version 3）Compose文件以集群模式部署时，将会忽略此选项。
> - `network_mode:"host"`不能与links混合。

#### networks

#### restart

`no`是默认的重启策略，它在任何情况下都不会重启容器。当指定`always`时，容器总是重新启动。如果退出码指示存在on-failure错误，则`on-failure`策略将重新启动容器。

```yaml
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

> **注意：**当使用（version 3）Compose文件以集群模式部署时，将会忽略此选项。使用[restart_policy](https://docs.docker.com/compose/compose-file/#restart_policy)代替

### networks

### volumes