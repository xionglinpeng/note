# dockerfile-maven-plugin

未完成：

1. 第六节：build命令行参数
2. 第六节：依赖于其他服务的docker镜像
3. 第七节：其他工具集成



This maven plugin integrates Maven with Docker.

dockerfile-maven-plugin插件是用于docker镜像打包，仓库上传的docker插件。它的前身是`docker-maven-plugin`，这个插件仍然是可用的，但是现在已经不推荐使用了，不推荐的原因即是设计目标。

设计目标：

- 不做任何花哨的事情。Dockerfile是构建docker镜像的方式，这是强制的。
- Make the Docker build process integrate with the Maven build process. If you bind the default phases, when you type `mvn package`, you get a Docker image. When you type `mvn deploy,` your image gets pushed.
- Make the goals remember what you are doing. You can type `mvn dockerfile:build` and later `mvn dockerfile:tag` and later `mvn dockerfile:push` without problems. This also eliminates the need for something like `mvn dockerfile:build -DalsoPush`; instead you can just say `mvn dockerfile:build dockerfile:push`.
- Integrate with the Maven build reactor. You can depend on the Docker image of one project in another project, and Maven will build the projects in the correct order. This is useful when you want to run integration tests involving multiple services.

## 1、Set-up

This plugin requires Java 7 or later and Apache Maven 3 or later (dockerfile-maven-plugin <=1.4.6 needs Maven >= 3, and for other cases, Maven >= 3.5.2). To run the integration tests or to use the plugin in practice, a working Docker set-up is needed.

## 2、Depedency

首先需要添加插件依赖。

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.13</version>
    <executions>
        <execution>
            <id>default</id>
            <goals>
                <goal>build</goal>
                <goal>push</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!--Dockerfile文件所在目录-->
        <contextDirectory>${project.basedir}</contextDirectory>
        <!--镜像仓库，镜像名称-->
        <repository>empty-window</repository>
        <!--镜像标签-->
        <tag>${project.version}</tag>
        <!--镜像构建参数-->
        <buildArgs>
            <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

## 3、Usage

### 3.1、Setting Environment

添加环境变量

```shell
DOCKER_HOST=tcp://<host>:2375
```

如果不指定，则默认地址为`tcp://localhost:2375`。

> 注意：如果使用IntelliJ IDEA开发工具，在配置完环境变量之后需要重新启动IntelliJ IDEA，使其重新加载环境变量。

### 3.2、Build Args

关于`buildArgs`参数，Dockerfile看起来应该是像这样的

```dockerfile
FROM openjdk:8-jre
MAINTAINER David Flemström <dflemstr@spotify.com>

ENTRYPOINT ["/usr/bin/java", "-jar", "/usr/share/myservice/myservice.jar"]

# Add Maven dependencies (not shaded into the artifact; Docker-cached)
ADD target/lib           /usr/share/myservice/lib
# Add the service itself
ARG JAR_FILE
ADD target/${JAR_FILE} /usr/share/myservice/myservice.jar
```

### 3.3、Consistent build lifecycle

构建的声明周期如下：

```shell
$ mvn package
$ mvn dockerfile:build
$ mvn verify
$ mvn dockerfile:push
$ mvn deploy
```

使用时，只需要

```shell
$ mvn deploy
```

### 3.4、Maven Goals

This plugin has 4 goals:

1. `dockerfile:build`

     构建镜像，前提是Dockerfile文件依赖的资源必须存在，比如jar包需要已经打包好了。Maven命令如下：

     ```shell
     $ mvn dockerfile:build
     ```

2. `dockerfile:help`
     Display help information on dockerfile-maven-plugin.
     Call `mvn dockerfile:help -Ddetail=true -Dgoal=<goal-name>` to display parameter
     details.

3. `dockerfile:push`

     向远处仓库推送镜像，前提是镜像必须存在。Maven命令如下：

     ```shell
     $ mvn dockerfile:push
     ```

4. `dockerfile:tag`

     ```shell
     $ mvn dockerfile:tag
     ```

## 4、Skip Docker Goals Bound to Maven Phases

可以将如下选项传递给Maven来跳过Goals。

| Maven Option            | What Does it Do?                                             | Default Value |
| ----------------------- | ------------------------------------------------------------ | ------------- |
| `dockerfile.skip`       | Disables the entire dockerfile plugin; all goals become no-ops. | false         |
| `dockerfile.build.skip` | Disables the build goal; it becomes a no-op.                 | false         |
| `dockerfile.tag.skip`   | Disables the tag goal; it becomes a no-op.                   | false         |
| `dockerfile.push.skip`  | Disables the push goal; it becomes a no-op.                  | false         |

For example, to skip the entire dockerfile plugin:
```
mvn clean package -Ddockerfile.skip
```

## 5、Features

1. 如果整个镜像（包括Dockerfile文件，依赖的jar包，脚本文件等）没有发生任何变化，重复构建更新镜像不会被更新。如果发送了变化，新的镜像将被构建，而旧的镜像会将其`REPOSITORY`和`TAG`置为`<none>`。

2. 使用`mvn dockerfile:build`构建完成镜像之后，将会在项目的`taraget`目下生成`docker`目录，此目录中包含四个文件，分别是`image-id`、`image-name`、`repository`、`tag`：

   ```
   .
   └── target
       └── docker
           ├── image-id
           ├── image-name
           ├── repository
           └── tag
   ```

## 6、Configuration

| Maven Option                      | What Does it Do?                                             | Required | Default Value |
| --------------------------------- | ------------------------------------------------------------ | -------- | ------------- |
| `dockerfile.contextDirectory`     | Dockerfile所在目录。                                         | yes      | none          |
| `dockerfile.repository`           | 镜像仓库，也是镜像名称。                                     | no       | none          |
| `dockerfile.tag`                  | 镜像标签。                                                   | no       | latest        |
| `dockerfile.buildArgs`            | 自定义构建参数。                                             | no       | none          |
| `dockerfile.build.noCache`        | 在构建映像时不使用缓存。                                     | no       | false         |
| `dockerfile.build.pullNewerImage` | Updates base images automatically.                           | no       | true          |
| `dockerfile.build.cacheFrom`      | Docker image used as cache-from. Pulled in advance if not exist locally or `pullNewerImage` is `false` | no       | none          |
| `dockerfile.build.squash`         | Squash newly built layers into a single new layer (experimental API 1.25+). | no       | false         |


配置参数详细说明：

- `dockerfile.contextDirectory`：Dockerfile所在目录。如果不指定`contextDirectory`，则默认`Dockerfile`文件位于项目的根目录下，如果不存在项目根目录下，则会报错：`Missing Dockerfile in context directory: D:\workspace\{project-name}`。

  对应`pom.xml`下`<configuration>`标签下的`<contextDirectory>`标签，如果同时指定了`<contextDirectory>`标签和命令行参数，优先以`<contextDirectory>`标签生效。

  ```shell
  $ mvn dockerfile:build -Ddockerfile.contextDirectory=${project.basedir}
  ```

- `dockerfile.repository`：镜像仓库，也是镜像名称。

  对应`pom.xml`下`<configuration>`标签下的`<repository>`标签，如果同时指定了`<contextDirectory>`标签和命令行参数，优先以`<repository>`标签生效。

  ```shell
  $ mvn dockerfile:build -Ddockerfile.repository=registry.cn-hangzhou.aliyuncs.com/xionglinpeng/${project.artifactId}
  ```

- `dockerfile.tag`：镜像标签。

  对应`pom.xml`下`<configuration>`标签下的`<tag>`标签，如果同时指定了`<tag>`标签和命令行参数，优先以`<tag>`标签生效。

  ```shell
  $ mvn dockerfile:build -Ddockerfile.tag=latest
  ```

- `dockerfile.build.noCache`：在构建映像时不使用缓存。在Features第一个特性时已经说过，其就是这个这个参数进行控制的，将其设置为true，每次构建时都是一个新的镜像，无论是否发生变化。

  ```shell
  $ mvn dockerfile:build -Ddockerfile.build.noCache=true
  ```

- `dockerfile.buildArgs`



## 7、Depend on Docker images of other services

## 8、Use other Docker tools that rely on Dockerfiles

## 9、Authentication

### 9.1、Authenticating with maven settings.xml

从1.3.6版开始，可以通过Maven setting.xml文件配置镜像仓库身份验证，只需要在setting.xml文件文件下配置如下内容：

```xml
<servers>
    <server>
        <id>registry.cn-hangzhou.aliyuncs.com</id>
        <username>repoUserName</username>
        <password>repoPassword</password>
    </server>
</servers>
```

然后在Maven pom.xml配置`<useMavenSettingsForAuth>`标签，如下：

```xml
<configuration>
    <repository>registry.cn-hangzhou.aliyuncs.com/xionglinpeng/${project.artifactId}</repository>
    <tag>${project.version}</tag>
    <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
</configuration>
```

或者通过命令行：

```shell
mvn dockerfile:push -Ddockerfile.useMavenSettingsForAuth=true
```

> 注意：
>
> `<server>`标签下`<id>`标签的值必须与`<configuration>`标签下`<repository>`标签中镜像仓库的主机和端口（`[REGISTRY_HOST[:REGISTRY_PORT]`）一致，否则，将无效。

### 9.2、Authenticating with maven pom.xml

从1.3.XX版开始，可以通过Maven pom.xml文件配置镜像仓库身份验证，只需要在`<configuration>`标签中添加`<username>`和`<password>`即可，如下所示：

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>${version}</version>
    <configuration>
        <username>repoUserName</username>
        <password>repoPassword</password>
        <repository>${docker.image.prefix}/${project.artifactId}</repository>
        <buildArgs>
            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

通过命令行：

```shell
$mvn dockerfile:push -Ddockerfile.username={username} -Ddockerfile.password={password}
```

## 10、Github

dockerfile-maven → Github：https://github.com/spotify/dockerfile-maven

docker-maven-plugin → Github：https://github.com/spotify/docker-maven-plugin