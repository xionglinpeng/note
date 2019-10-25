## 1.1. How to Include Feign

要在项目中包含Feign，请使用包含group`org.springframework.cloud` 和artifact id`spring-cloud-starter-openfeign`的starter。有关使用当前的Spring Cloud发布系列设置构建系统的详细信息，请参阅

Maven

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.1.3.RELEASE</version>
</dependency>
```

Gradle

```groovy
compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-openfeign', version: '2.1.3.RELEASE'
```

*Example spring boot app*

```java
@SpringBootApplication
@EnableFeignClients
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**StoreClient.java**

```java
@FeignClient("stores")
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    List<Store> getStores();

    @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
    Store update(@PathVariable("storeId") Long storeId, Store store);
}
```

在`@FeignClient`注解中，上面例子中字符串（"stores"）是任意一个微服务客户端的名称，用于创建一个Ribbon负载均衡器。还可以使用URL属性（角色值或主机名）指定URL。Feign接口在应用程序上下文中bean的名称是接口的完全限定名。如果想要指定自己的别名值，可以使用`@FeignClient`注解的`qualifier`属性。

Ribbon客户端希望发现“stores”服务的物理地址。如果您的应用程序是一个Eureka客户端，那么它将在Eureka服务注册表中解析服务。如果您不想使用使用Eureka，您可以简单地在外部配置中配置一个服务列表（URL属性）。

## 1.2. Overriding Feign Defaults

Spring Cloud's Feign支持的一个核心概念是named client。每个Feign客户端是一组组件的一部分，这些组件协同工作以根据需要联系远程服务器。作为一个使用`@FeignClient`注解的应用程序开发人员，这个ensemble有一个名字。Spring Cloud使用`FeignClientsConfiguration`根据需要为每个指定的客户端创建一个新的ensemble作为一个`ApplicationContext`。这个包含一个`feign.Decoder`, `feign.Encoder`和 `feign.Contract`。可以使用`@FeignClient`注解的`contextId`属性来该覆盖ensemble的名称。

通过使用`@FeignClient`声明额外的配置（在FeignClientsConfiguration之上），Spring Cloud允许您完全控制feign客户端。示例：

```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    //..
}
```

在这种情况下，客户端是由`FeignClientsConfiguration`中已经存在的组件和FooConfiguration中的任何组件（其中后者将覆盖前者）组成的。





`name`和`url`属性支持占位符（placeholder）。

```java
@FeignClient(name = "${feign.name}", url = "${feign.url}")
public interface StoreClient {
    //..
}
```

Spring Cloud Netflix默认为feign提供一下bean（）：

## 1.7. Feign Inheritance Support

Feign通过单继承接口支持样板api。这允许将通用操作分组到方便的基本接口中。

**UserService.java**

```java
public interface UserService {

    @RequestMapping(method = RequestMethod.GET, value ="/users/{id}")
    User getUser(@PathVariable("id") long id);
}
```

**UserController.java**

```java
@RestController
public class UserController implements UserService {

}
```

**UserClient.java**

```java
package project.user;

@FeignClient("users")
public interface UserClient extends UserService {

}
```

> 通常不建议在服务端和客户端之间共享接口。它进入了紧密耦合，而且在Spring MVC当前形式（也许未来可以）下也不能工作（方法参数映射不是继承的）。

## 1.9. Feign logging

为每一个Feign客户端创建一个日志记录器。默认情况下，logger的名称是用于创建Feign客户端接口的全限定类名。Feign只响应`DEBUG`级别的日志。

**application.properties**

```properties
logging.level.project.user.UserClient=DEBUG
```

您可以为每个客户端配置`Logger.Level`对象，告诉Feign要记录什么多少日志。可选项：

- `NONE`：没有日志（DEFAULT）。
- `BASIC`：只记录**请求方法**、**URL**以及**响应状态码**和**执行时间**。
- `HEADERS`：记录基本的信息以及request和response头。
- `FULL`：记录headers，body以及请求和响应元数据。

例如，下面将把`Logger.Level`设置为`FULL`：

```java
@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

## 1.10. Feign `@QueryMap` Support

OpenFeign`@QueryMap`注解支持将POJO对象用作GET请求的参数映射。不幸的是，默认的OpenFeign的@QueryMap注解与Spring不兼容，因为它缺少`value`属性。

Spring Cloud OpenFeign提供了一个等价的`@SpringQueryMap`注解用于将POJO或Map参数映射为查询参数。

例如，`Params`类定义参数`param1`和`param2`：

```java
// Params.java
public class Params {
    private String param1;
    private String param2;

    // [Getters and setters omitted for brevity]
}
```

下面的Feign客户端使用`@SpringQueryMap`注解来使用`Params`类：

```java
@FeignClient("demo")
public interface DemoTemplate {

    @GetMapping(path = "/demo")
    String demoEndpoint(@SpringQueryMap Params params);
}
```

如果你需要对生成的查询参数映射进行更多控制，可以实现一个定制的`QueryMapEncoder`Bean。

































