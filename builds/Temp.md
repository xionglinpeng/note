# dockerfile-maven-plugin

This maven plugin integrates Maven with Docker.

dockerfile-maven-plugin插件是用于docker镜像打包，仓库上传的docker插件。它的前身是`docker-maven-plugin`，这个插件仍然是可用的，但是现在已经不推荐使用了，不推荐的原因即是设计目标。

设计目标：

- 不做任何花哨的事情。Dockerfile是构建docker镜像的方式，这是强制的。
- Make the Docker build process integrate with the Maven build process. If you bind the default phases, when you type `mvn package`, you get a Docker image. When you type `mvn deploy,` your image gets pushed.

## Set-up

This plugin requires Java 7 or later and Apache Maven 3 or later (dockerfile-maven-plugin <=1.4.6 needs Maven >= 3, and for other cases, Maven >= 3.5.2). To run the integration tests or to use the plugin in practice, a working Docker set-up is needed.

## Depedency

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

## Usage

### Setting Environment

添加环境变量

```shell
DOCKER_HOST=tcp://<host>:2375
```

如果不指定，则默认地址为`tcp://localhost:2375`。

> 注意：如果使用IntelliJ IDEA开发工具，在配置完环境变量之后需要重新启动IntelliJ IDEA，使其重新加载环境变量。

### Build Args

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

### Maven Goals

This plugin has 4 goals:

1. `dockerfile:build`
2. `dockerfile:help`
     Display help information on dockerfile-maven-plugin.
     Call `mvn dockerfile:help -Ddetail=true -Dgoal=<goal-name>` to display parameter
     details.
3. `dockerfile:push`
4. `dockerfile:tag`

## Features

1. 如果整个镜像（包括Dockerfile文件，依赖的jar包，脚本文件等）没有发生任何变化，重复构建更新镜像不会被更新。如果发送了变化，新的镜像将被构建，而旧的镜像会将其`REPOSITORY`和`TAG`置为`<none>`。

2. 使用`mvn dockerfile:build`构建完成镜像之后，将会在项目的`taraget`目下生成`docker`目录，此目录中包含四个文件，分别是`image-id`、`image-name`、`repository`、`tag`：

   ```
   |-target/
   |---docker/
   |-----image-id
   |-----image-name
   |-----repository
   |-----tag
   ```

## Configuration





| Maven Option         | What Does it do?                                             | required | Default Value |
| -------------------- | ------------------------------------------------------------ | -------- | ------------- |
| contextDirectory     | Dockerfile所在目录。                                         | YES      | none          |
| repository           | 镜像仓库，也是镜像名称。                                     | NO       | none          |
| tag                  | 镜像标签。                                                   | NO       | latest        |
| buildArgs            | 自定义构建参数                                               | NO       | none          |
| build.pullNewerImage | 自动更新基础镜像。                                           | NO       | true          |
| build.noCache        | ??在构建映像时不要使用缓存。                                 | NO       | false         |
| build.cacheFrom      | ??用作缓存的Docker图像。提前拉，局部不存在或pullNewerImage为假 | NO       | none          |
| build.squash         | ??将新构建的层压缩成一个新层(实验API 1.25+)。                | NO       | false         |

配置参数详细说明：

- `contextDirectory`：Dockerfile所在目录。如果不指定`contextDirectory`，则默认`Dockerfile`文件位于项目的根目录下，如果不存在项目根目录下，则会报错：`Missing Dockerfile in context directory: D:\workspace\{project-name}`。

## Github

dockerfile-maven → Github：https://github.com/spotify/dockerfile-maven

docker-maven-plugin → Github：https://github.com/spotify/docker-maven-plugin





未完成：

1. 设计目标
2. Maven相关名称
3. build配置
4. 仓库推送
5. 其他工具集成



# Usage

## Maven Goals

Goals available for this plugin:

| Goal               | Description                              | Default Phase |
| ------------------ | ---------------------------------------- | ------------- |
| `dockerfile:build` | Builds a Docker image from a Dockerfile. | package       |
| `dockerfile:tag`   | Tags a Docker image.                     | package       |
| `dockerfile:push`  | Pushes a Docker image to a repository.   | deploy        |

### Skip Docker Goals Bound to Maven Phases

You can pass options to maven to disable the docker goals.

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

## Configuration

### Build Phase

| Maven Option                      | What Does it Do?                                             | Required | Default Value |
| --------------------------------- | ------------------------------------------------------------ | -------- | ------------- |
| `dockerfile.contextDirectory`     | Directory containing the Dockerfile to build.                | yes      | none          |
| `dockerfile.repository`           | The repository to name the built image                       | no       | none          |
| `dockerfile.tag`                  | The tag to apply when building the Dockerfile, which is appended to the repository. | no       | latest        |
| `dockerfile.build.pullNewerImage` | Updates base images automatically.                           | no       | true          |
| `dockerfile.build.noCache`        | Do not use cache when building the image.                    | no       | false         |
| `dockerfile.build.cacheFrom`      | Docker image used as cache-from. Pulled in advance if not exist locally or `pullNewerImage` is `false` | no       | none          |
| `dockerfile.buildArgs`            | Custom build arguments.                                      | no       | none          |
| `dockerfile.build.squash`         | Squash newly built layers into a single new layer (experimental API 1.25+). | no       | false         |