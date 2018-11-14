# Spring Boot

## 配置SSL

首先我们需要一个秘钥证书库，可以使用JDK的keytool工具生成，如下：

```shell
C:\Users\xlp>keytool -genkeypair -alias root -keypass 597646251 -storepass 597646251 -storetype PKCS12 -keyalg RSA -validity 3600 -keystore "root.p12" -dname "CN=LINPENG.XIONG, OU=存在信号, O=存在信号, L=成都市, ST=四川省, C=zh_CN"
```

执行之后会产生一个名为`root.p12`秘钥库文件，将此文件导入到项目目录下，可以导入到项目根目录下，也可以导入到项目classpath目录下。

然后添加相应的配置开启SSL，如下：

```properties
spring.application.name=spring-ssl

server.port=8443
# 开启SSL
server.ssl.enabled=true
# 指定秘钥库位置
server.ssl.key-store=classpath:root.p12
# 指定秘钥库中其中一个条目的别名
server.ssl.key-alias=root
# 秘钥库的口令
server.ssl.key-store-password=597646251
# 秘钥口令
server.ssl.key-store-type=PKCS12
# 秘钥库提供者，使用keytool工具生成的一般是SUN
server.ssl.key-store-provider=SUN
```

注意：

> 秘钥库如果是放在classpath路径，那么需要指定秘钥库为`classpath:root.p12`。如果秘钥库是放在项目根目录下，那么直接指定相对位置`server.ssl.key-store=root.p12`即可，默认为从根目录开始查找。

配置完成之后我们需要编写一个接口进行测试，如下：

```java
@GetMapping("/hello")
@ResponseBody
public Object hello() {
    return "Hello SSL.";
}
```

启动服务，可以发现控制台答应了如下内容：

```
Tomcat initialized with port(s): 8443 (https)
......
Tomcat started on port(s): 8443 (https) with context path ''
```

先使用http访问/hello接口：http://localhost:8443/hello，结果返回了如下错误信息：

```
Bad Request
This combination of host and port requires TLS.
```

再使用HTTPS访问：https://localhost:8443/hello，结果提示了如下错误信息：

```
localhost 使用了不受支持的协议。
```

这是因为我们在生成秘钥库的时候没有指定秘钥的算法，即`-keyalg`，这里需要指定为`-keyalg RSA`，如果不知道，默认为`DSA`。



> 关于SSL证书以及秘钥库的相关信息这里不做描述。
>
> 在做测试的时候，将根证书导入可信任的，不知道为什么，仍然不能使我们自签名的证书可信任。
>
> 如果以后解决了，可以在此记录笔记。

## 端点管理

#### HTTP管理服务器(ManagementServerProperties)

HTTP管理服务器定义了端点的访问地址

```properties
# 管理端点应该绑定到的网络地址，如果将端口设置为-1，则表示禁用所有端点。
management.server.address=127.0.0.1
# 端点管理服务端口号，默认以应用端口一致。
management.server.port=20002
# 端点管理服务上下文路径，默认“/”。
management.server.servlet.context-path=/management
# 在每个响应中添加“X-Application-Context”HTTP报头。
management.server.add-application-context-header=false
```



```properties
# 默认情况下是否启用或禁用所有端点，默认“true”。
management.endpoints.enabled-by-default=true
```



#### 端点WEB配置 (WebEndpointProperties)

端点WEB配置定义了端点的访问路径

```properties
# 端点管理基础路径，默认“/actuator。
management.endpoints.web.base-path=/actuator
# 自定义端点的路径。
management.endpoints.web.path-mapping.[endpoint]=/path
# 包含的web端点，默认“health,info”，如果需要包含全部，设置“*”。
management.endpoints.web.exposure.include=health,info
# 排除的web端点，如果需要排除全部，设置“*”。
management.endpoints.web.exposure.exclude=shutdown
```



```properties
# 健康检查详情，可选值：always、never、when_authorized。
management.endpoint.health.show-details=always
```

### 自定义健康检查

自定义一个健康检查器，继承`org.springframework.boot.actuate.health.AbstractHealthIndicator`，并将自定义的健康检查器Bean添加到IOC容器中。

健康检查器，是由健康检查端点（`/actuator/health`）访问展示，默认是不展示详情的，需要添加`management.endpoint.health.show-details=always`。

```java
import org.springframework.boot.actuate.health.AbstractHealthIndicator;
import org.springframework.boot.actuate.health.Health;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator extends AbstractHealthIndicator {

    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        builder.up().withDetail("MyHealthIndicator","Day Day Up");
    }
}
```

