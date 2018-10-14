

# Spring-Cloud-Config-Server



服务端

添加依赖

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

在启动类上添加注解，开启配置服务

```java
@EnableConfigServer
```



```properties
spring.application.name=star-travel-guest-config-server
server.port=8888
# git仓库位置（可以指定本地文件系统或实际的git地址）
spring.cloud.config.server.git.uri=file:///D:/program/**/**...
spring.cloud.config.server.git.uri=https://github.com/**/**...
# 配置仓库路径下的相对搜索位置
spring.cloud.config.server.git.search-paths=
# 访问git仓库用户名
spring.cloud.config.server.git.username=************@163.com
# 访问git仓库密码
spring.cloud.config.server.git.password=************
# 从git仓库克隆下来的配置文件本地存储位置
spring.cloud.config.server.git.basedir=D:/program/**/**...
```



{label}：对应git分支，默认master





客户端

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```





新建`bootstrap.properties`或`bootstrap.yml`配置文件

添加如下配置

```properties
spring.cloud.config.enabled=true
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.service-id=star-travel-guest-config-server
# 配置中心服务的地址，默认：“http://localhost:8888”。
spring.cloud.config.uri=http://localhost:8888
# 对应配置文件规则中的{application}。
spring.cloud.config.name=star-travel-guest-destination
# 对应配置文件规则中的{profile}，默认“default”。
spring.cloud.config.profile=default
# 对应配置文件规则中的{label}，默认“master”。
spring.cloud.config.label=master
```



注意

> 客户端的配置必须写在`bootstrap.properties`或`bootstrap.yml`配置文件中。

## 基础架构

客户端应用从配置管理中获取配置信息遵从下面的执行流程：

1. 应用启动时，根据`bootstrap.properties`中配置的应用名`{applicaton}`、环境名`{profile}`、分支名`{label}`，向Config Server请求获取配置信息。
2. Config Server根据自己维护的Git仓库信息和客户端传递过来的配置定位信息去查找配置信息。
3. 通过`git clone`命令将找到的配置信息下载到Config Server的文件系统中。
4. Config Server创建Spring的`ApplicationContext`实例，并从Git背地仓库中加载配置文件，最后将这些配置内容读取出来返回给客户端应用。
5. 客户端应用在获得外部配置文件后加载到客户端的`ApplicationContext`实例，该配置内容的优先级高于客户端Jar包内部的配置内容，所以在Jar包中重复的内容将不再被加载。

Config Server巧妙地通过`git clone`将配置信息存于背地，起到了缓存的作用，即使当Git服务端无法访问的时候，依然可以取Config Server中的缓存内容进行使用。

## 服务端详解


### Git配置仓库



```properties
spring.cloud.config.server.git.uri=https://github.com/xxx/xxx-config[/][.git]
```



#### 占位符配置URL

{application}、{profile}、{label}这些占位符除了用于标识配置文件的规则之外，还可以用于Config Server中对Git仓库地址的URI配置。

#### 配置多个仓库



#### 子目录存储

Config Server可以将配置文件定位到Git仓库的子目录中。

```properties
spring.cloud.config.server.git.search-paths=/a/b/c
```

`spring.cloud.config.server.git.search-paths`参数配置也支持使用`{application}`、`{profile}`、`{label}`占位符。

例如：

```properties
spring.cloud.config.server.git.search-paths={application}
```

- `{application}`：对应客户端`spring.cloud.config.name`配置。
- `{profile}`：对应客户端`spring.cloud.config.profile`配置。
- `{label}`：对应客户端`spring.cloud.config.label`配置。

#### 访问权限

Git仓库一般都是需要账号密码才能访问，配置如下：

```properties
spring.cloud.config.server.git.uri=https://github.com/**/**...
# 访问git仓库用户名
spring.cloud.config.server.git.username=************@163.com
# 访问git仓库密码
spring.cloud.config.server.git.password=************
```

### 本地仓库

使用Git或SVN仓库，会在本地文件系统中存储一份，默认存储在`file:/C:/Users/xlp/AppData/Local/Temp/config-repo-14843937889943659909/`临时目录。由于其随机性以及临时目录的特性，可能会有一些不可预知的后果，为了避免将来可能会出现的问题，最好的办法就是指定一个固定的位置来存储这些重要的信息。可以使用如下配置指定：

路径可以指定系统环境变量，表达式为`${}`，例如`${user.dir}`，表示当前项目所在目录。

```properties
spring.cloud.config.server.git.basedir=${user.dir}/star-travel-guest-config
```

### 本地文件系统

Spring Cloud Config提供了本地文件系统的存储方式来保存配置信息。只需要设置配置属性

```properties
spring.profiles.active=native
```

Config Server会默认从应用的`src/main/resource`目录下搜索配置文件。如果需要指定搜索配置文件的路径。可以通过如下属性指定：

```properties
spring.cloud.config.server.native.search-locations=
```

### 属性覆盖



### 健康监测

### 安全保护

由于配置中心存储的内容比较敏感，做一点的安全处理是必需的。为配置中心实现安全保护的方式有很多，比如物理网络限制、OAuth2授权等。不过，由于我们的微服务应用和配置中都构建于Spring Boot基础上，所以与Spring     Security结合使用会更加方便。

首先添加依赖

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

添加之后就可以不做任何操作了，Spring Security自动配置会自动创建一个默认用户名为`user`，密码为随机生成的UUID（启动控制台会打印）。

一般情况下我们不会使用默认生成的，可以自己指定，Config Server配置如下：

```properties
spring.security.user.name=user
spring.security.user.password=0310d588-6ad8-452a-a44a-9713ebca20d1
spring.security.user.roles=admin
```

Config Client就需要配置其访问账户密码，如下：

```properties
spring.cloud.config.username=user
spring.cloud.config.password=0310d588-6ad8-452a-a44a-9713ebca20d1
```

### 加密解密

在Spring Cloud Config中通过在属性值前使用`{cipher}`前缀来标注该内容是一个加密值，当微服务客户端加载配置时，配置中心会自动为带有`{cipher}`前缀的值进行解密。比如下面的例子：

```properties
spring.datasource.password={cipher}0028cd5d522499d795c6cd236a374bc1e58600c6c384c2853349100188bb5f8a
```

#### 相关端点

`org.springframework.cloud.config.server.encryption.EncryptionController`

```properties
[GET]:[/encrypt/status]
[GET]:[/key/{name}/{profiles}]
[GET]:[/key]
[POST]:[/encrypt]
[POST]:[/encrypt/{name}/{profiles}]
[POST]:[/decrypt]
[POST]:[/decrypt/{name}/{profiles}]
```

#### 对称加密

- 前提

  为了启用该功能，需要在配置中心的运行环境中安装布线长度的JCE（Java Cryptography Extension）版本。虽然，JCE功能在JRE中自带，但是默认使用的是有长度限制的版本。

  **下载Java Cryptography Extension (JCE) **

  [download JCE](https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)。

  解压之后文件如下：

  ```
  local_policy.jar
  README.txt
  US_export_policy.jar
  ```

  **安装JCE**

  将`local_policy.jar`和`US_export_policy.jar`拷贝至`jre/lib/security`目录下。

  安装完成之后启动Config Server。

  请求端点`/encrypt/status`：

  ```shell
  [root@localhost ~] curl http://localhost:8888/encrypt/status
  {"description":"No key was installed for encryption service","status":"NO_KEY"}
  ```

  返回了如上信息，其已经明确说明了“加密服务没有安装密钥”，所以下一步就是配置秘钥。

- 配置秘钥

  此秘钥是进行加密的秘钥（对称性秘钥）。

  **注意：**必须配置在`bootstrap.properties`以及更高优先级的配置中。

  ```properties
  encrypt.key=7d9b280a-edc8-4a07-b0cd-65fbab04a102
  ```

  启动服务，再次请求端点`/encrypt/status`，信息如下：

  ```json
  [root@localhost ~] curl http://localhost:8888/encrypt/status
  {"status":"OK"}
  ```

- 使用`/encrypt`端点和`/decrypt`端点加解密

  **注意：**它们都是`POST`请求

  ```bash
  [root@localhost ~] curl http://localhost:8888/encrypt -d root
  0028cd5d522499d795c6cd236a374bc1e58600c6c384c2853349100188bb5f8a
  
  [root@localhost ~] curl http://localhost:8888/decrypt -d 0028cd5d522499d795c6cd236a374bc1e58600c6c384c2853349100188bb5f8a
  root
  ```

> 测试的环境为JDK10，没有安装JCE，加密对称加密功能仍然可用。查阅到的资料是需要安装，可能是不同的版本的差异，也可能使使用了有限长度的默认JCE。

#### 非对称加密





### 高可用配置

高可用即部署了多个配置服务，可以通过外部应用负载均衡访问，例如Nginx。

不过这里我们可以直接以服务发现的方式实现客户端负载均衡去访问，可以使用eureke、zookeeper...等等服务注册中心。

主要是要将配置服务注册到服务中心，客户端进行服务发现，客户端需要如下配置：

```properties
# 开启通过服务来访问Config Server的功能
spring.cloud.config.discovery.enabled=true
# Config Server注册的服务名
spring.cloud.config.discovery.service-id=star-travel-guest-config-server
```

## 客户端详解

### URI指定配置中心

Spring Cloud Config的客户端在启动的时候，默认会从工程的classpath中加载配置信息并启动应用。只有当我们配置`spring.cloud.config.uri`的时候，客户端应用才会尝试连接Spring Cloud Config的服务端来获取远程配置信息并初始化Spring环境配置。同时，我们必须将该参数配置在bootstrao.properties、环境变量或是其他优先级高于应用Jar包内的配置信息中，才能正确加载到远程配置。若不指定`spring.cloud.config.uri`参数的话，Spring Cloud Config的客户端会默认尝试连接``

### 服务化配置中心





### 失败快速响应与重试

#### 失败快速响应

Spring Cloud Config的客户端会预先加载很多其他信息，然后再开始连接Config Server进行属性的注入，当我们构建的应用较为复杂的时候，可能在连接Config Server之前花费较长的启动时间，而在一些特殊场景下，我们又希望可以快速知道当前应用示顺序地从Config Server获取到配置信息，这对在初期构建调试环境时，可以减少很多等待启动的时间。

要实现上述功能只需要在`bootstrap.properties`配置文件中添加如下配置：

```properties
# 默认值false
spring.cloud.config.fail-fast=true
```

在添加和不添加的情况下启动服务，可以发现，添加了配置之后，就会报如下错误；而不添加时，在报错之前已经加载了很多其他的内容。

```java
java.lang.IllegalStateException: Could not locate PropertySource and the fail fast property is set, failing
```

#### 失败快速重试

`spring.cloud.config.fail-fast`配置了快速响应失败，但是，如果只是因为网络波动等其他间歇性原型导致的问题，直接启动失败似乎代价有些高。所以，Config客户端还提供了自动重试的功能，在开启重试功能之前，先确保已经配置了`spring.cloud.config.fail-fast=true`。

只需要在添加`spring-retry`、`spring-boot-starter-aop`已经即可：

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

```

再次启动客户端查看日志，可以看到如下日志打印了6次：

```java
[  restartedMain] c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://localhost:8888
[  restartedMain] c.c.c.ConfigServicePropertySourceLocator : Connect Timeout Exception on Url - http://localhost:8888. Will be trying the next url if available
```

重试自定义配置

```properties
# 初始重试间隔时间（单位为毫秒），默认为1000毫秒
spring.cloud.config.retry.initial-interval=1000
# 下一间隔的乘数，默认为1.1，所以当最初间隔为1000毫秒时，下一次失败后的间隔为1100毫秒
spring.cloud.config.retry.multiplier=1.1
# 最大重试次数，默认为6次
spring.cloud.config.retry.max-attempts=6
# 最大间隔时间，默认为2000毫秒
spring.cloud.config.retry.max-interval=200
```

### 获取远程配置

通过URL请求和客户端配置的访问对应总结如下：

- 通过向Config Server发送GET请求以直接的方式获取。

  **Resource**

  `org.springframework.cloud.config.server.resource.ResourceController`

  ```properties
  [GET]:[/{name}/{profile}/{label}/**],produces=[application/octet-stream]
  [GET]:[/{name}/{profile}/{label}/**]
  [GET]:[/{name}/{profile}/**],params=[useDefaultLabel]
  ```

  **Environment**

  `org.springframework.cloud.config.server.environment.EnvironmentController`

  ```properties
  [GET]:[/{name}-{profiles}.properties]
  [GET]:[/{name}-{profiles}.yml || /{name}-{profiles}.yaml]
  [GET]:[/{label}/{name}-{profiles}.json]
  [GET]:[/{name}/{profiles:.*[^-].*}]
  [GET]:[/{label}/{name}-{profiles}.yml || /{label}/{name}-{profiles}.yaml]
  [GET]:[/{label}/{name}-{profiles}.properties]
  [GET]:[/{name}-{profiles}.json]
  [GET]:[/{name}/{profiles}/{label:.*}]
  ```

- 通过客户端配置的方式加载。

  ```properties
  # 对应配置文件的{name}
  spring.cloud.config.name=application
  # 对应配置文件的{profiles}
  spring.cloud.config.profile=dev
  # 对应配置文件的{label}
  spring.cloud.config.label=master
  ```

### 动态刷新配置

客户端添加`actuator`依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

开启`refresh`端点管理

```properties
management.endpoints.web.exposure.include=refresh
```

添加`@RefreshScope`注解

```java
package com.star.travel.guest.destination.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope
public class Controller {
    
    @Value("${name}")
    private String name;

    @RequestMapping("/name")
    public Object name(){
        return name;
    }
}
```

测试：

有配置`name=success`

1. 首先先请求`http://localhost:8080/name`，返回结果`success`。

2. 请求`http://localhost:8080/actuator/refresh`端点，结果是`[]`。

3. 修改配置为`name=success-2018`。

4. 再次请求`http://localhost:8080/name`，结果仍然是`success`。

5. 请求`http://localhost:8080/actuator/refresh`端点，结果是`[“name”]`。

6. 再次请求`http://localhost:8080/name`，结果是`success-2018`。

**注意：**1.x.x版本的Spring Boot和2.x.x版本的Spring Boot是有差异的，上述测试例子是基于2.x.x版本的Spring Boot。

> 该功能还可以同Git仓库的Web Hook功能进行关联，，当有Git提交变化时，就给对应的配置主机发送/actuator/refresh请求来实现配置信息的实时更新。但是，当我们的系统发展壮大之后，维护这样的刷新清单也将成为一个非常大的负担，而且很容易犯错，那么就可以通过Spring Cloud Bus来实现以消息总线的方式进行配置变更的通知，并完成集群上的批量配置更新。

## BUG

1. 测试版本`Finchley.SR1`。使用高可用配置时，Config Client虽然能发现配置服务，但是Config Client本身却不能被注册为服务。
2. `Dalston.SR3`、`Dalston.SR2`版本不能对配置文件加密，若需要调整到`Dalston.SR1`或者期待`Dalston.SR4`的发布。