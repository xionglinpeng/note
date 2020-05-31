# Apollo

使用apollo注意事项

1. metaservice、configservice、eurekaservice在逻辑上是三个不同的模块，但是在实际的代码中是同一个服务。

2. 注册中心使用的是eureka，为了适配非java语言服务的分布式配置，所以抽象了metaservice服务以获取对应的configservice和adminservice服务实例。metaservice等价于是eurekaservice代理服务，其本质上就是metaservice从eurekaservice上获取对应的服务实例。

3. DEV环境，SIT环境，UAT环境与PROD环境不太可能使用同一套apollo配置中心（DEV环境的操作将配置中心搞挂了怎么办，岂不是会影响生产环境），所以一般情况下每个环境都部署各自的一套。而对于portal来说却又不太适合各自部署一套（如果有4个环境，岂不是有4个配置界面，操作管理太麻烦了）。所以portal只需要部署一套，就可管理所有的环境（portal作为一个管理端，挂了也不会影响生产环境）。为了能管理所有环境，需要能发现每个环境adminservice服务的地址，第二步已经说过，需要通过metaservice获取，而metaservice与configservice是同一个服务，所以进而需要指定每一个环节的configservice的地址，通过`{environment}-meta=http://configservice-url`指定（其实这里应该指定的是metaservice服务的地址，但是metaservice与configservice是同一个服务）。

4. portal从metaservice获取到adminservice服务的地址，进而通过adminservice管理配置，所以需要保证adminservice注册到eureka的服务地址可被portal访问。

   如果不能被访问，地址网络配置方式：

   *eureka.instance.ip-address*

   ```properties
   # jvm虚拟机变量
   -Deureka.instance.ip-address=${指定IP}
   # 系统环境变量
   EUREKA_INSTANCE_IP_ADDRESS=${指定IP}
   # bootstrap.yml配置文件
   eureka:
     instance:
       ip-address: ${指定的IP}
   ```

   *eureka.instance.homePageUrl*

   ```properties
   # jvm虚拟机变量
   -Deureka.instance.homePageUrl=http://${指定IP}:${指定PORT}
   # 系统环境变量
   EUREKA_INSTANCE_HOME_PAGE_URL=http://${指定IP}:${指定PORT}
   # bootstrap.yml配置文件
   eureka:
     instance:
       homePageUrl: http://${指定的IP}:${指定的Port}
       preferIpAddress: false
   ```

5. 需要保证adminservice能够与configservice通信，因为eurekaservice集成到了configservice，而adminservice也需要注册到注册中心。

   有四种配置方式（每种方式可以通过jvm虚拟机变量，系统环境变量，bootstrap.yml配置文件配置）：

   下面的四种配置方式优先级以此降低，即`eureka.service.url`优先级最高，`hostname`最低。

   1. eureka.service.url

      这个配置位于数据库`ApolloConfigDB#ServerConfig`表中，存在属性eureka.service.url，默认值为`http://localhost:8080/eureka`

   2. eureka.client.serviceUrl.defaultZone

      默认位于配置文件`bootstrap.yml`，默认值为`http://${eureka.instance.hostname}:8080/eureka`

      ```properties
      # jvm虚拟机变量
      -Deureka.client.serviceUrl.defaultZone=http://${指定IP}:${指定PORT}/eureka
      # 系统环境变量
      EUREKA_CLIENT_SERVICE_URL_DEFAULT_ZONE=http://${指定IP}:${指定PORT}/eureka
      # bootstrap.yml配置文件
      eureka:
        client:
          serviceUrl: 
            defaultZone: http://${指定的IP}:${指定的Port}/eureka
      ```

   3. eureka.instance.hostname

      默认位于配置文件`bootstrap.yml`，默认值为`${hostname:localhost}`

      ```properties
      # jvm虚拟机变量
      -Deureka.instance.hostname=${服务实例IP或域名}
      # 系统环境变量
      EUREKA_INSTANCE_HOSTNAME=${服务实例IP或域名}
      # bootstrap.yml配置文件
      eureka:
        instance:
          hostname: ${服务实例IP或域名}
      ```

   4. hostname

      ```properties
      # jvm虚拟机变量
      -Dhostname=${服务实例IP或域名}
      # 系统环境变量
      HOSTNAME=${服务实例IP或域名}
      # bootstrap.yml配置文件
      hostname: ${服务实例IP或域名}
      ```

6. apollo日志记录在`/opt/logs/10000317[1|2|3]/apollo-[configservice|adminservice|portal].log`文件中（可通过项目的`application.yml`文件查看和更改），所以请确保`/opt/logs/`有写权限。



## 本地启动

apollo-configservice，adminservice和portalservice都有各自的启动类，不过apollo提供了一个apollo-assembly模块可以启动所有这三服务，当然，其本质都是一样的。

**configservice和adminservice**

*指定服务标识*

```shell
--configservice --adminservice
```

*指定profile*

profile都统一指定github，这是因为在`application-github.properties`文件中指定数据库连接信息的配置，这个文件位于`apollo-common`模块下的`src/main/resources`目录中。

```properties
-Dapollo_profile=github
APOLLO_PROFILE=github

# 虚拟机参数
-Dspring.profile.active=github
# 系统环境变量
SPRING_PROFILE_ACTIVE=github
# bootstrap.yml配置文件
spring:
  profile:
    active: github
```

指定数据库连接信息

根据`application-github.properties`文件声明的配置指定即可。

```properties
# 虚拟机参数
-Dspring_datasource_url=
-Dspring_username_url=
-Dspring_password_url=
```



**portalservice**

*指定服务标识*

```shell
--portal
```

指定配profile



指定数据库连接信息



*指定{environment}-meta地址*

```properties
-Ddev_meta=http://${configservice_url}:8080
DEV_META=http://${configservice_url}:8080
```













## 源码构建

apollo的源码构建很简单，其源码中已经提供了构建脚本，位于`apollo-master/scripts/build.[bat.sh]`，只需要执行这个脚本即可完成构建。

打开脚本，你可能会注意到脚本声明了对数据库连接信息以及metaservice地址的配置选项，这里不推荐修改这些内容，因为这些信息可以通过命令行参数，环境变量等方式传入，这样只需要一次打包，就可以在所有环境都使用。

至于更具体的信息，请参考官方文档。

## docker部署

1. 编译Apollo源码，直接运行Apollo源码中的scripts/build.[sh|bat]脚本即可构建。构建完成之后可以在target目录下看到如下内容：

   ├── apollo
   │   ├── apollo-adminservice
   │   │   ├── apollo-adminservice-1.7.0-SNAPSHOT-github.zip
   │   │   └── Dockerfile
   │   ├── apollo-configservice
   │   │   ├── apollo-configservice-1.7.0-SNAPSHOT-github.zip
   │   │   └── Dockerfile
   │   ├── apollo-portal
   │   │   ├── apollo-portal-1.7.0-SNAPSHOT-github.zip
   │   │   └── Dockerfile
   │   └── docker-compose.yaml



<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
</plugin>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.2.0</version>
</plugin>





2. 将构建好的zip文件以及源码中Dockerfile文件组装成如下结构

```shell
├── apollo
│   ├── apollo-adminservice
│   │   ├── apollo-adminservice-1.7.0-SNAPSHOT-github.zip
│   │   └── Dockerfile
│   ├── apollo-configservice
│   │   ├── apollo-configservice-1.7.0-SNAPSHOT-github.zip
│   │   └── Dockerfile
│   ├── apollo-portal
│   │   ├── apollo-portal-1.7.0-SNAPSHOT-github.zip
│   │   └── Dockerfile
│   └── docker-compose.yaml
```

3. 进入apollo目录，构建镜像

```shell
$ docker build -t apollo-adminservice:1.7.0-SNAPSHOT-alpine ./apollo-adminservice
$ docker build -t apollo-configservice:1.7.0-SNAPSHOT-alpine ./apollo-configservice
$ docker build -t apollo-portal:1.7.0-SNAPSHOT-alpine ./apollo-portal
```

4. 标签镜像，用于将其发布到阿里云

```shell
$ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/[命名空间]/[仓库]:[镜像版本号]

$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/xionglinpeng/apollo-adminservice:1.7.0-SNAPSHOT-alpine
$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/xionglinpeng/apollo-configservice:1.7.0-SNAPSHOT-alpine
$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/xionglinpeng/apollo-portal:1.7.0-SNAPSHOT-alpine
```

5. 发布到阿里云

```shell
$ docker push registry.cn-hangzhou.aliyuncs.com/xionglinpeng/apollo-adminservice:1.7.0-SNAPSHOT-alpine
$ docker push registry.cn-hangzhou.aliyuncs.com/xionglinpeng/apollo-configservice:1.7.0-SNAPSHOT-alpine
$ docker push registry.cn-hangzhou.aliyuncs.com/xionglinpeng/apollo-portal:1.7.0-SNAPSHOT-alpine
```

6. 创建docker-compose.yaml

```yaml
version: "3.7"
services:
  apollo-configservice:
    image: registry.cn-hangzhou.aliyuncs.com/xionglinpeng/apollo-configservice:1.7.0-SNAPSHOT-alpine
    container_name: apollo-configservice
    ports:
      - "30080:8080"
    restart: always
    environment: # 注意数据库连接中的&字符需要转义
      DS_URL: "jdbc:mysql://[host]:3306/ApolloConfigDB?characterEncoding=utf8\\&useSSL=false"
      DS_USERNAME: "root"
      DS_PASSWORD: "[password]"
    networks:
      - network-apollo
    volumes:
      - volume-apollo:/opt/logs/
  apollo-adminservice:
    image: registry.cn-hangzhou.aliyuncs.com/xionglinpeng/apollo-adminservice:1.7.0-SNAPSHOT-alpine
    container_name: apollo-adminservice
    ports:
      - "30090:8090"
    restart: always
    environment: # 注意数据库连接中的&字符需要转义
      DS_URL: "jdbc:mysql://[host]:3306/ApolloConfigDB?characterEncoding=utf8\\&useSSL=false"
      DS_USERNAME: "root"
      DS_PASSWORD: "[password]"
      EUREKA_INSTANCE_HOSTNAME: apollo-configservice # adminservice需要注册到eureka
    networks:
      - network-apollo
    volumes:
      - volume-apollo:/opt/logs/
  apollo-portal:
    image: registry.cn-hangzhou.aliyuncs.com/xionglinpeng/apollo-portal:1.7.0-SNAPSHOT-alpine
    container_name: apollo-portal
    ports:
      - "30070:8070"
    restart: always
    environment: # 注意数据库连接中的&字符需要转义
      DS_URL: "jdbc:mysql://[host]:3306/ApolloPortalDB?characterEncoding=utf8\\&useSSL=false"
      DS_USERNAME: "root"
      DS_PASSWORD: "[password]"
      DEV_META: "http://apollo-configservice:8080" # 配置metaservice地址
    networks:
      - network-apollo
    volumes:
      - volume-apollo:/opt/logs/
networks:
  network-apollo:
    external: true
volumes:
  volume-apollo:
    external: true
```

7. 启动apollo

   1. 创建网络

      ```shell
      $ docker network create network-apoll
      ```

   2. 创建volume

      ```shell
      $ docker volume create volume-apoll
      ```

   3. 运行mysql

   4. 启动apollo

      ```shell
      $ docker-compose -f docker-compose.yaml up -d
      ```

      