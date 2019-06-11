# Spring Cloud Netflix Zuul





## 集成Swagger2

需求描述：Swagger的集成本身是很简单的，那么这里为什么到单独说到Swagger与Zuul的集成呢？

swagger本身只针对集成的应用本身，而在微服务中，不同的微服务都有自己的API，那么将会导致如下两个问题：

- 接口都是统一通过zuul网关暴露，而具体微服务接口被封闭，即使集成了Swagger，也不能访问其API文档。
- 即使将具体微服务的接口开放，但是多个微服务就会有多个文档地址，不方便管理。

所以，我们需要的是统一通过网关服务的Swagger文档就可以访问到所有微服务的Swagger API文档。

其关键的核心在于Swagger提供的`springfox.documentation.swagger.web.SwaggerResourcesProvider`接口和`springfox.documentation.swagger.web.SwaggerResource`类。

**添加依赖**

首先需要添加swagger依赖。

Note：zuul网关和具体的微服务都需要集成Swagger。

```xml
<!-- swagger2 -->
<dependency>
    <groupId>com.spring4all</groupId>
    <artifactId>swagger-spring-boot-starter</artifactId>
    <version>1.8.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-annotations</artifactId>
    <version>1.5.22</version>
</dependency>
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-models</artifactId>
    <version>1.5.22</version>
</dependency>
```

**配置Swagger Resource**

在网关服务配置Swagger资源。

原理：

所有具体的微服务文档所应用的Swagger静态资源都是集成在网关（zuul）服务中的Swagger静态资源，只是通过`/v2/api-docs`接口获取对应的文档数据。

配置示例如下：

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;
import springfox.documentation.swagger.web.SwaggerResource;
import springfox.documentation.swagger.web.SwaggerResourcesProvider;

import java.util.ArrayList;
import java.util.List;

/**
 * TODO notes : spring cloud zuul with swagger2 integration.
 */
@Configuration
public class Swagger2Configuration {

    @Component
    @Primary
    public static class DocumentationConfig implements SwaggerResourcesProvider {

        @Override
        public List<SwaggerResource> get() {
            List<SwaggerResource> resources = new ArrayList<>();
            resources.add(swaggerResourceLocation("GATEWAY","/v2/api-docs","1.0.0"));
            resources.add(swaggerResourceLocation("ORG","/org/v2/api-docs","1.0.0"));
            resources.add(swaggerResourceUrl("SYS",
                                       "http://localhost:8083/v2/api-docs","2.3.3"));
            resources.add(swaggerResourceUrl("API",
                                       "http://localhost:8082/v2/api-docs","2.3.3"));
            return resources;
        }

        private SwaggerResource swaggerResourceLocation(
            						String name,String location,String version){
            SwaggerResource resource = new SwaggerResource();
            resource.setName(name);
            resource.setLocation(location);
            resource.setSwaggerVersion(version);
            return resource;
        }

        private SwaggerResource swaggerResourceUrl(
            						String name,String url,String version){
            SwaggerResource resource = new SwaggerResource();
            resource.setName(name);
            resource.setUrl(url);
            resource.setSwaggerVersion(version);
            return resource;
        }
    }
}
```

## 跨域



```properties
zuul.sensitive-headers=Access-Control-Allow-Origin
zuul.ignored-headers=Access-Control-Allow-Origin
```





```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsFilterConfiguration {

    @Bean
    public CorsFilter corsFilter(){
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.addAllowedOrigin("*");
        configuration.addAllowedHeader("*");
        configuration.addAllowedMethod("*");
        configuration.addExposedHeader("token");
        configuration.addExposedHeader("TOKEN");
        source.registerCorsConfiguration("/**",configuration);
        return new CorsFilter(source);
    }
}

final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		final CorsConfiguration config = new CorsConfiguration();
		config.setAllowCredentials(true); // 允许cookies跨域
		config.addAllowedOrigin("*");// 允许向该服务器提交请求的URI，*表示全部允许。。这里尽量限制来源域，比如http://xxxx:8080 ,以降低安全风险。。
		config.addAllowedHeader("*");// 允许访问的头信息,*表示全部
		config.setMaxAge(18000L);// 预检请求的缓存时间（秒），即在这个时间段里，对于相同的跨域请求不会再预检了
		config.addAllowedMethod("*");// 允许提交请求的方法，*表示全部允许，也可以单独设置GET、PUT等
    /*    config.addAllowedMethod("HEAD");
        config.addAllowedMethod("GET");// 允许Get的请求方法
        config.addAllowedMethod("PUT");
        config.addAllowedMethod("POST");
        config.addAllowedMethod("DELETE");
        config.addAllowedMethod("PATCH");*/
		source.registerCorsConfiguration("/**", config);

```

