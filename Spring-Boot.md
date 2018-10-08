# Spring Boot

## 端点管理

#### HTTP管理服务器(ManagementServerProperties)

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

