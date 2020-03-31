# Dockerfile







## CMD

CMD指令用来指定**启动容器时**默认执行的命令。它支持三种格式：

- `CMD command param1 param2`在/bin/sh中执行，提供给需要交互的应用；
- `CMD ["executable","param1","param2"]`使用exec执行，是推荐使用的方式；
- `CMD ["param1","param2"]`提供给`ENTRYPOINT`的默认参数。

CMD指令提供容器运行时的默认值，这些默认值可以是一条指令，也可以是一些参数。一个Dockerfile中可以有多条CMD指令，但是只有最后一条CMD指令有效。

如果用户在命令行界面运行docker run命令时指定了命令参数，则会覆盖CMD指令中的命令。

`CMD ["param1","param2"]`格式是在CMD指令和ENTRYPOINT指令配合时使用的，CMD指令中的参数会添加到ENTRYPOINT指令中。

使用shell和exec格式时，命令在容器中的运行方式与RUN指令相同。不同之处在于，RUN指令是在构建镜像时执行命令，并生成新的镜像；CMD指令在构建镜像时不执行任何命令，而是在容器启动时默认将CMD指令作为第一条执行的命令。

**示例：**

*测试命令行参数*

```shell
[root@localhost test]# docker run alpine:3.11.3 echo "hello alpine"
hello alpine
```

*创建Dockerfile*

```dockerfile
FROM alpine:3.11.3
MAINTAINER XLP
CMD echo "hello docker"
```

*构建镜像并运行测试*

```shell
[root@localhost test]# docker build -t my-alpine .
[root@localhost test]# docker run my-alpine
hello docker
```

*再次测试命令行参数*

```shell
[root@localhost test]# docker run my-alpine echo "hello my-alpine"
hello my-alpine
```

**exec格式测试**

*编辑dockerfile*

```dockerfile
FROM alpine:3.11.3
MAINTAINER XLP
CMD ["echo","my name is alpine:3.11.3"]
```

*重新构建并运行测试*

```shell
[root@localhost test]# docker build -t my-alpine .
[root@localhost test]# docker run my-alpine
my name is alpine:3.11.3
[root@localhost test]# docker run my-alpine echo "hello"
hello
```

## ENTRYPOINT

指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执行，所有传入值将作为该命令的参数。

ENTRYPOINT指令有两种格式：

1. `ENTRYPOINT command param1 param2`（shell格式）
2. `ENTRYPOINT  ["executable,"param1","param2"]`（exec格式，推荐格式）

一个Dockerfile中可以有多条ENTRYPOINT指令，但只有最后一条ENTRYPOINT指令有效。

在运行时，可以被`--entrypoint`参数覆盖掉，如`docker run --entrypoint`。

CMD指令指定值将作为根命令的参数。

当使用Shell格式时，ENTRYPOINT指令会忽略任何CMD指令和`docker run`命令的参数，并且会运行在`/bin/sh -c`中。这意味着ENTRYPOINT指令进程为`/bin/sh -c`的子进程，进程在容器中的PID将不是1，且不能接受Unix信号。即当使用`docker stop <container>`命令时，命令进程接收不到`SIGTERM`信号。

推荐使用exec格式，使用此格式时，docker run传入的命令参数会覆盖CMD指令的内容并且附加到ENTRYPOINT指令的参数中。从ENTRYPOINT的使用中可以看出，CMD可以是参数，也可以是指令，而ENTRYPOINT只能是命令；另外docker run命令提供的运行命令参数可以覆盖CMD，但不能覆盖ENTRYPOINT。

**示例：**

*创建dockerfile*

```dockerfile
FROM alpine:3.11.3
MAINTAINER XLP
CMD ["hello","alpine"]
ENTRYPOINT ["echo"]
```

*构建镜像*

```shell
[root@localhost test]# docker build -t my-alpine .
```

*运行测试*

```shell
# 测试运行
[root@localhost test]# docker run my-alpine
hello alpine      #相当于执行了echo "hello alpine"

# 测试命令行参数覆盖CMD指令声明的参数
[root@localhost test]# docker run my-alpine "I'm docker by alpine"
I'm docker by alpine

# 测试--entrypoint选项覆盖ENTRYPOINT指令
[root@localhost test]# docker run --entrypoint "cat"  my-alpine "/etc/hosts"
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	eb43593a3324
```

