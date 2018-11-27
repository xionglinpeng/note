# Spring Boot

## [IV. Spring Boot features](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features)

### [23. SpringApplication](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-spring-application)

#### [23.1. Startup Failure](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-startup-failure)

如果应用程序无法启动，注册的FailureAnalyzers就有机会提供专门的错误消息和修复问题的具体操作。例如，如果您在端口8080上启动web应用程序，并且该端口已经在使用，您应该会看到类似于以下消息的内容:

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

![](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/images/note.png) Spring Boot提供了许多FailureAnalyzer实现，您可以添加自己的。

如果没有故障分析程序能够处理异常，您仍然可以显示完整的条件报告，以便更好地理解出了什么问题。为此，你需要启用DEBUG属性或者DEBUG日志：`org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener`。

例如，如果您使用java -jar运行您的应用程序，那么您可以启用debug属性如下:

```shell
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```



#### [23.2. Customizing the Banner](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-banner)

可以通过向类路径添加`banner.txt`文件或将`spring.banner.location`属性设置为此类文件的位置来更改在启动时打印的banner。

| Variable                           | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| `${application.version}`           | 您的应用程序的版本号，如MANIFEST.MF中声明的那样。例如，实现版本:1.0被打印为1.0。 |
| `${application.formatted-version}` |                                                              |
| `${spring-boot.version}`           |                                                              |
| `${spring-boot.formatted-version}` |                                                              |
|                                    |                                                              |
| `${application.title}`             |                                                              |

document sources: 

https://www.cnblogs.com/cc11001100/p/7456145.html

https://www.breakyizhan.com/springboot/3249.html

https://www.degraeve.com/img2txt.php

                       .::::.
                     .::::::::.
                    :::::::::::
                 ..:::::::::::'
              '::::::::::::'
                .::::::::::
           '::::::::::::::..
                ..::::::::::::.
              ``::::::::::::::::
               ::::``:::::::::'        .:::.
              ::::'   ':::::'       .::::::::.
            .::::'      ::::     .:::::::'::::.
           .:::'       :::::  .:::::::::' ':::::.
          .::'        :::::.:::::::::'      ':::::.
         .::'         ::::::::::::::'         ``::::.
     ...:::           ::::::::::::'              ``::.
    ```` ':.          ':::::::::'                  ::::..
                       '.:::::'                    ':'````..
**佛祖**

                            _ooOoo_
                           o8888888o
                           88" . "88
                           (| -_- |)
                           O\  =  /O
                        ____/`---'\____
                      .'  \\|     |  `.
                     /  \\|||  :  |||  \
                    /  _||||| -:- |||||-  \
                    |   | \\\  -  / |   |
                    | \_|  ''\---/''  |   |
                    \  .-\__  `-`  ___/-. /
                  ___`. .'  /--.--\  `. . __
               ."" '<  `.___\_<|>_/___.'  >'"".
              | | :  `- \`.;`\ _ /`;.`/ - ` : | |
              \  \ `-.   \_ __\ /__ _/   .-` /  /
         ======`-.____`-.___\_____/___.-`____.-'======
                            `=---='
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                      佛祖保佑, 永无BUG !
**神兽**

```
          ┏┛ ┻━━━━━┛ ┻┓
          ┃　　　　　　 ┃
          ┃　　　━　　　┃
          ┃　┳┛　  ┗┳　┃
          ┃　　　　　　 ┃
          ┃　　　┻　　　┃
          ┃　　　　　　 ┃
          ┗━┓　　　┏━━━┛
            ┃　　　┃   神兽保佑
            ┃　　　┃   代码无BUG！
            ┃　　　┗━━━━━━━━━┓
            ┃　　　　　　　    ┣┓
            ┃　　　　         ┏┛
            ┗━┓ ┓ ┏━━━┳ ┓ ┏━┛
              ┃ ┫ ┫   ┃ ┫ ┫
              ┗━┻━┛   ┗━┻━┛
          
　　　　　　　 ┏┓       ┏┓+ +
　　　　　　　┏┛┻━━━━━━━┛┻┓ + +
　　　　　　　┃　　　　　　 ┃
　　　　　　　┃　　　━　　　┃ ++ + + +
　　　　　　 █████━█████  ┃+
　　　　　　　┃　　　　　　 ┃ +
　　　　　　　┃　　　┻　　　┃
　　　　　　　┃　　　　　　 ┃ + +
　　　　　　　┗━━┓　　　 ┏━┛
               ┃　　  ┃
　　　　　　　　　┃　　  ┃ + + + +
　　　　　　　　　┃　　　┃　Code is far away from     bug with the animal protecting
　　　　　　　　　┃　　　┃ + 　　　　         神兽保佑,代码无bug
　　　　　　　　　┃　　　┃
　　　　　　　　　┃　　　┃　　+
　　　　　　　　　┃　 　 ┗━━━┓ + +
　　　　　　　　　┃ 　　　　　┣┓
　　　　　　　　　┃ 　　　　　┏┛
　　　　　　　　　┗┓┓┏━━━┳┓┏┛ + + + +
　　　　　　　　　 ┃┫┫　 ┃┫┫
　　　　　　　　　 ┗┻┛　 ┗┻┛+ + + +
```









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

