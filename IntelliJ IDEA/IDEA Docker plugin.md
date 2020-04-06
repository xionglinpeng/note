# IDEA Docker plugin

提供与Docker的集成

- 下载并构建Docker镜像
- 从拉取的镜像或直接从Dockerfile创建和运行Docker容器
- 使用专用的Docker运行配置
- 使用Docker Compose运行多容器应用程序

IDEA Plugin：https://plugins.jetbrains.com/plugin/7724-docker

## 1. 安装docker插件

在File—>Settings—>Plugins搜索Docker，安装，然后重启IDEA。

![1586150833587](images\1586150833587.png)

## 2. 配置Docker

在File—>Settings—>Build,Execution,Deployment—>Docker配置连接Docker守护进程，如果连接成功，将会显示Connection successful。

![1586151313780](images\1586151313780.png)

> 1. Docker连接配置可以选择Docker Machine或者Docker Engine API URL。
> 2. Docker连接配置可以配置多个不同的Docker守护进程。

## 3. 连接Docker

通过IDEA工具栏View—>Tool Windows—>Docker打开窗口：

![1586152069571](images\1586152069571.png)

点击Connect连接Docker：

![1586152097844](images\1586152097844.png)

连接之后就可以看到镜像和容器了。

![1586152267423](images\1586152267423.png)

##  4. 编写Dockerfile

![1586152494836](images\1586152494836.png)

##  5. 配置镜像构建与容器部署方式

当编写好Dockerfile文件之后就可以通过直接运行Dockerfile文件构建镜像与容器，当初次针对一个新的项目是需要先配置，如下图所示，点击Edit 'xxx'：

![1586152599443](images\1586152599443.png)

显示界面如下，可以配置镜像的构建以及容器的部署方式。

1. 首先选择对应的Server，即在第二步的配置。
2. 之后选择Dockerfile文件，因为是通过Dockerfile文件进入到此界面，所以它自己已经填上了。
3. Context folder —> 指定当前配置所对应的上下文。
4. 配置镜像的构建方式，如下界面所示：
   1. Image tag：镜像标签
   2. Build args：构建参数
   3. Run built image：控制是否构建镜像的同时并运行其容器。
   4. Container name：容器名称
   5. Executable：
      - Entrypoint：
      - Command：
   6. Publish exposed ports to the host interfaces：
      - All：随机端口
      - Specify：指定端口
        - Bind ports：绑定的端口
   7. Bind mounts：挂载的数据卷
   8. Environment variables：环境变量
   9. Command line options：启动命令

![1586152630077](images\1586152630077.png)

配置完成之后可以通过`Command preview`查看将会执行的命令。

确认无误之后直接点击`Run`即可构建镜像并运行容器。

当下一次再需要构建镜像并运行容器的时候，就需要再次配置了，直接点击`Run 'xxx'`运行即可。

## 6. 构建并部署

点击`Run 'xxx'`构建镜像并部署容器

运行成功，可以通过`Log`菜单容器服务的运行日志，界面如下：

![1586155129173](images\1586155129173.png)

运行失败，可以通过`Deploy log`查看失败的原因，界面如下：

![1586155195673](images\1586155195673.png)

> 运行一次之后，构建了一个镜像和容器，当修改了代码需要再次构建镜像并部署容器时，可以直接运行，Docker插件将会删除旧的镜像和容器，重新构建新的。

## 7. 容器运行菜单

容器运行成功之后提供了一些管理功能，可以方便开发：

- Deploy log：容器部署日志。

- Log：容器服务运行日志。

- Attached console：附加控制台。

- Properties：容器的基本信息，包括当前容器所对应的镜像ID，容器ID，容器名称等。

- Environment variables：容器的环境变量。

- Port Bindings：容器映射的端口。

- Volume Bindings：容器绑定的数据卷。



> 注意：
>
> 在第二步的时候配置了连接到Docker守护进程的Docker Engine API URL，如果你的docker守护进程和IDEA是在同一个宿主机，直接配置连接地址即可。但是如果docker守护进程和IDEA不是在同一个宿主机，那么是不能直接连接的，因为Docker默认只支持本地IP连接，例如`127.0.0.1`。所以为了能够连接上，需要将其开放，配置如下：
>
> 1. 修改docker.service文件，添加监听端口 -H tcp://0.0.0.0:2375
>
> ```shell
> $ vim /usr/lib/systemd/system/docker.service
> ```
>
> 2. 添加-H tcp://0.0.0.0:2375
>
> ```shell
> ......
> ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375
> ......
> ```
>
> 3. 重启Docker守护进程
>
> ```shell
> $ systemctl restart docker
> ```
>
> *Windows开启*
>
> 在电脑左下角找到docker图标，鼠标右键选择settings。 将General菜单下的`Expose daemon on tcp://localhost:2375 without TLS`勾选。无需重启。
>
> ![](images\1586157553955.png)



## Maven插件：docker-maven-plugin

Maven的`docker-maven-plugin`插件也可以实现这个功能

```xml
<!--使用docker-maven-plugin插件-->
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.0.0</version>

    <!--将插件绑定在某个phase执行-->
    <executions>
        <execution>
            <id>build-image</id>
            <!--将插件绑定在package这个phase上。也就是说，
                用户只需执行mvn package ，就会自动执行mvn docker:build-->
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>

    <configuration>
        <!--指定生成的镜像名,这里是我们的项目名-->
        <imageName>${project.artifactId}</imageName>
        <!--指定标签 这里指定的是镜像的版本，我们默认版本是latest-->
        <imageTags>
            <imageTag>latest</imageTag>
        </imageTags>
        <!-- 指定我们项目中Dockerfile文件的路径-->
        <dockerDirectory>${project.basedir}/src/main/resources</dockerDirectory>

        <!--指定远程docker 地址-->
        <dockerHost>http://127.0.0.1:2375</dockerHost>

        <!-- 这里是复制 jar 包到 docker 容器指定目录配置 -->
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <!--jar包所在的路径  此处配置的即对应项目中target目录-->
                <directory>${project.build.directory}</directory>
                <!-- 需要包含的 jar包 ，这里对应的是 Dockerfile中添加的文件名　-->
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```