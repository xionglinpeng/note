# Spring Security实战



## Quick Start

### 搭建授权服务

添加jar包依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.3.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security.oauth.boot</groupId>
    <artifactId>spring-security-oauth2-autoconfigure</artifactId>
</dependency>
```

注解启用授权服务

```java
@EnableAuthorizationServer
```

添加配置

```properties
server.port=8090
# 声明一个用户
spring.security.user.name=admin
spring.security.user.password=123456
# 声明一个客户端（可以理解为为客户颁发一个用户名和密码）
security.oauth2.client.client-id=client
security.oauth2.client.client-secret=c97cdc0f-7e8b-56d3-9bd5-db178579945a
# 开放token和token检查接口
security.oauth2.authorization.token-key-access=permitAll()
security.oauth2.authorization.check-token-access=permitAll()
```

### 搭建资源服务

注解启用资源服务

```java
@EnableResourceServer
```

添加配置

```properties
server.port=8099
# 声明客户端的的id和secret（可以理解为访问授权服务时的用户名和密码）
security.oauth2.client.client-id=client
security.oauth2.client.client-secret=c97cdc0f-7e8b-56d3-9bd5-db178579945a
# 声明验证token有效性的URL地址
security.oauth2.resource.token-info-uri=http://localhost:8090/oauth/check_token
```



















security启动时输出的日志信息-过滤器链

```
o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: any request, [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@5f08fe00, org.springframework.security.web.context.SecurityContextPersistenceFilter@5a566922, org.springframework.security.web.header.HeaderWriterFilter@7a1a3468, org.springframework.security.web.authentication.logout.LogoutFilter@184823ed, org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationProcessingFilter@5aa62ee7, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@42ed89da, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@7135ce0a, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@a1691c0, org.springframework.security.web.session.SessionManagementFilter@467045c4, org.springframework.security.web.access.ExceptionTranslationFilter@5d352de0, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@3fdcde7a]
```





```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
```













