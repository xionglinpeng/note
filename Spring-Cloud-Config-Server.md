

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

#### Encryption

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





### 服务化配置中心





### 失败快速响应与重试



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





## BUG

1. 使用高可用配置时，Config Client虽然能发现配置服务，但是Config Client本身却不能被注册为服务？