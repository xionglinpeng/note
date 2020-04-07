# Dockerfile

Dockerfile是Docker用来构建镜像的文本文件，包括自定义的指令和格式。可以通过`docker build`命令从Dockerfile中构建镜像。用户可以通过统一的语法命令来根据需求进行配置，通过这份统一的配置文件，在不同的系统上进行分发，需要使用时就可以根据配置文件进行自动化构建，这解决了开发人员构建镜像的复杂过程。

Dockerfile描述了组装对象的步骤，其中每条指令都是单独运行的。除了`FROM`指令，其他每条命令都会在上一条指令所生成镜像的基础上执行，执行完后会生成一个新的镜像层，新的镜像层覆盖在原来的镜像之上，从而形成了新的镜像。Dockerfile所生成的最终镜像就是基础镜像上面叠加一层层的镜像层组建的。

**Docker指令**

在Dockerfile中，指令不区分大小写，但是为了与参数区分，推荐大写。

Docker会顺序执行Dockerfile中的指令，第一条指令必须是`FROM`指令，它用于指定构建镜像的基础镜像。在Dockerfile中以`#`开头的行是注释，而在其他位置出现的`#`会被当成参数。

Dockerfile中的指令有`FROM`、`MAINTAINER`、`USER`、`LABEL`、`ARG`、`WORKDIR`、`ENV`、`RUN`、`ADD`、`COPY`、`EXPOSE`、`VOLUME`、`CMD`、`ENTRYPOINT`、`ONBUILD`、`HEALTHCHECK`、`SHELL`、`STOPSIGNAL`，错误的指令会被忽略。



## FROM

格式：`FROM <image>`或者`FROM <image>:<tag>`

`FROM`指令的功能是为后面的指令提供基础镜像，因此Dockerfile必须以FROM指令作为第一条非注释指令。从公共镜像库中拉取镜像很容易，基础镜像可以选择任何有效的镜像。

在一个Dockerfile中FROM指令可以出现多次，这样会构建多个镜像。`tag`的默认值是`latest`，如果参数image或者tag指定的镜像不存在，则返回错误。

## MAINTAINER？？



## LABEL??



## ARG??





## ENV

格式：`ENV <key> <value>`或者`ENV <key>=<value> ...`

`ENV`指令可以为镜像创建出来的容器声明环境变量。并且在Dockerfile中，ENV指令声明的环境变量会被后面的特定指令（即`ENV`、`ADD`、`COPY`、`WORKDIR`、`EXPOSE`、`VOLUME`、`USER`）解释使用。

其他指令使用环境变量时，使用格式为`$variable_name`或者`${variable_name}`，如果在变量面前添加斜杠\可以转义。如`\$foo`或者`\${foo}`将会被转换为`$foo`或者`${foo}`，而不是环境变量所保存的值。另外，`ONBUILD`指令不支持环境替换。

## COPY

格式：`COPY <src> <dest>`

`COPY`指令赋值所指向的文件或目录，将它添加到镜像中，复制的文件或目录在镜像中的路径是`<dest>`。`<src>`所指定的源可以有多个，但必须是上下文根目录中的相对路径。不能只用形如`COPY ../something /something`这样的指令。此外，`<src>`可以使用通配符指向所有匹配通配符的文件和目录，例如，`COPY home* /mydir/`表示添加所有以`home`开头的文件到目录`/mydir/`中。

`<dest>`可以是文件或目录，但必须是目标镜像中的绝对路径或者相对于`WORKDIR`的相对路径（）。若`<dest>`以反斜杠`/`结尾则其指向的是目录；否则指向文件。`<src>`同理。若`<dest>`是一个文件，则`<src>`的内容会被写到`<dest>`中；否则`<src>`指向的文件或目录中的内容会被赋值添加到`<dest>`目录中。

当`<src>`指定多个源时，`<dest>`必须是目录。如果`<dest>`不存在，则路径中不存在的目录会被创建。

## ADD

格式：`ADD <src> <dest>`

`ADD`与`COPY`指令在功能上很相似，都支持复制本地文件到镜像的功能，但`ADD`指令还支持其他功能。`<src>`可以是指向网络文件的URL，此时若`<dest>`指向一个目录，则URL必须是完全路径，这样可以获得网络文件的文件名filename，该文件会被复制添加到`<dest>/<filename>`。比如`ADD http://example.com/config.property / `会创建文件`/config.property`。

`<src>`还可以指向一个本地压缩归档文件，该文件会在复制到容器时会被解压提取，如`ADD example.tar.xz /`。但是若URL中的文件为归档文件则不会被解压提取。

`ADD`和`COPY`指令虽然功能相似，但一般推荐使用`COPY`，因为`COPY`只支持本地文件，相比`ADD`而言，它更加透明。

## EXPOSE

格式：`EXPOSE <port> [<port>/<protocol>...]`

EXPOSE指令通过Docker该容器在运行时侦听指定的网络端口。可以指定端口是侦听TCP还是UDP，如果未指定协议，则默认值为TCP。

这个指令仅仅是声明容器打算使用什么端口而已，并不会自动在宿主机进行端口映射，可以在运行的时候通过`docker -p`指定。

例如：

```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

## USER

格式：`USER <user>[:<group>]`或者`USER <UID>[:<GID>]`

`USER`指令设置了`user name`和`user group`（可选）。在它之后的`RUN`，`CMD`以及`ENTRYPOINT`指令都会以设置的user来执行。

## WORKDIR

`WORKDIR`指令设置工作目录，它之后的`RUN`、`CMD`、`ENTRYPOINT`、`COPY`以及`ADD`指令都会在这个工作目录下运行。如果这个工作目录不存在，则会自动创建一个。

`WORKDIR`指令可在`Dockerfile`中多次使用。如果提供了相对路径，则它将相对于上一个WORKDIR指令的路径的路径。

例如

```dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

输出结果是：/a/b/c。

## VOLUME？？

## RUN

格式1：`RUN <COMMAND>`（shell格式）

格式2：`RUN ["executable","param1","param2"]`（exec格式，推荐使用）

 RUN指令会在前一条创建出的镜像的基础上创建一个容器，并在容器中运行命令，在命令结束运行后提交容器作为新镜像，新镜像被Dockerfile中的下一条指令使用。

RUN指令的两种格式表示命令在容器中的两种运行方式。当使用shell格式时，命令通过`/bin/sh -c`运行。

当使用exec格式时，命令是直接运行的，容器不调用shell程序，即容器中没有shell程序。exec格式中的参数会被当成JSON数组被Docker解析，故必须使用双引号而不是使用单引号。因为exec格式不会在shell中执行，所以环境变量的参数不会被替换。

比如执行`RUN ["echo","$HOME"]`指令时，`$HOME`不会做变量替换。如果希望运行shell程序，指令可以写成`RUN ["/bin/bash","-c","echo","$HOME"]`。

## CMD

`CMD`指令用来指定**启动容器时**默认执行的命令。它支持三种格式：

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

## ONBUILD？？

## SHELL??

## HEALTHCHECK??

## STOPSIGNAL??

# Dockerfile实践指南

## 使用标签

给镜像打上标签，有利于帮助了解镜像的功能。

## 选择基础镜像

容器的镜像越小，越有利于迁移，而这跟选择的基础镜像有一定关系，不同的镜像大小不同，目前Linux镜像的大小有如下关系：

`busybox < alpine < debian < centos < ubuntu`

同时在构建自己的Docker镜像时，只安装和更新必要的包。对于基础的容器环境镜像，推荐使用alpine镜像，因为它非常轻量级（目前其大小只有5M左右），并且有其独有的apk包管理工具。

## 充分利用缓存

Docker deamon会顺序执行Dockerfile中的指令，而且一旦缓存失效，后续命令将不能使用缓存。为了有效地利用缓存，需要保证指令的连续性，尽量将所有Dockerfile文件相同的部分都放在前面，而将不同的部分放到后面。

## 正确使用`ADD`与`COPY`命令

当在Dockerfile中的不同部分需要用到不同的文件时，不要一次性将这些文件都添加到镜像中去。而是在需要时添加，这样也有利于重复利用docker缓存。

另外考虑到镜像大小问题，使用ADD指令去获取远程URL中的压缩包不是推荐的做法。应该使用`RUN wget`或`RUN curl`代替。这个样可以删除解压后不再需要的文件，并且不需要在镜像中添加一层。

错误做法：

```dockerfile
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

正确做法：

```dockerfile
RUN mkdir -p /usr/src/things \   
    && curl -SL http://example.com/big.tar.xz \    
    | tar -xJC /usr/src/things \    
    && make -C /usr/src/things all
```

## RUN指令

RUN指令指定的命令较长时可以使用反斜杠`\`分隔多行。

ubuntu系统下的包管理工具是apt-get，在该场景下有以下几点需要注意：

1. 







## `CMD`和`ENTRYPOINT`命令

`CMD`和`ENTRYPOINT`指令指定了容器运行的默认命令，推荐二者结合使用。使用`exec`格式的`ENTRYPOINT`指令设置固定的默认命令和参数，然后使用`CMD`指令设置可变的参数。

Dockerfile中的`CMD`和`ENTRYPOINT`指令顺序不重要（`CMD`写在`ENTRYPOINT`前后都可以）。

## 不要在Dockerfile中做端口映射

使用Dockerfile的`EXPOSE`指令，虽然可以将容器端口映射在主机端口上，但会破坏Dockerfile的可移植性，且这样的镜像在一台主机上只能启动一个容器。所以端口映射应在`docker run`命令中使用`-p`参数指定。

```dockerfile
# 不要在Dockerfile中做如下映射
EXPOSE 80:8080
# 仅暴露80端口，需要另做映射
EXPOSE 80
```





> 主要内容引用：<https://juejin.im/post/5e60745d518825492e4965ed#heading-13>





































