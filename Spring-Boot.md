# Spring Boot

## 配置SSL

```shell
C:\Users\xlp>keytool -genkeypair -alias root -keypass 597646251 -storepass 597646251 -storetype PKCS12 -validity 3600 -keystore "root.keystore" -dname "CN=LINPENG.XIONG, OU=存在信号, O=存在信号, L=成都市, ST=四川省, C=zh_CN"
```

将证书`root.keystore`添加到classpath下，添加如下配置：

```properties
spring.application.name=spring-ssl

server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:root.keystore
server.ssl.key-alias=root
server.ssl.key-store-password=597646251
server.ssl.key-store-type=PKCS12
server.ssl.key-store-provider=SUN
```

再写一个接口

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

