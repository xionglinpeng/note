----



## Security



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```



```properties
spring.security.user.name=user
spring.security.user.password=xxxxxx
```







## Email notification



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```



```properties
spring.mail.host=smtp.qq.com
spring.mail.username=296007576@qq.com
spring.mail.password=odybgnkqxxbpbjcg
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.properties.mail.smtp.ssl.enable=true
spring.mail.port=465
# 是谁发送出去的
spring.boot.admin.notify.mail.from=296007576@qq.com
# 发送给谁
spring.boot.admin.notify.mail.to=xionglinpeng@163.com

```



- `spring.boot.admin.notify.mail.enabled`

  Description : Enable mail notifications.

  Default Value : true

- `spring.boot.admin.notify.slack.ignore-changes`

  Description : Comma-delimited list of status changes to be ignored. Format: "<from-status>:<to-status>". Wildcards allowed.

  Default Value : `"UNKNOWN:UP"`

- `spring.boot.admin.notify.mail.from`

  Description : Mail sender

  Default Value : `Spring Boot Admin <noreply@localhost>`

- `spring.boot.admin.notify.mail.to`

  Description : Comma-delimited list of mail recipients

  Default Value : `"root@localhost"`

- `spring.boot.admin.notify.mail.cc`

  Description : Comma-delimited list of carbon-copy recipients

  Default Value : `[]`

- `spring.boot.admin.notify.mail.subject`

  Description : Mail subject. SpEl-expressions are supported.

  Default Value : `"#{application.name} (#{application.id}) is #{to.status}"`

- `spring.boot.admin.notify.mail.text`

  Description : Mail body. SpEL-expressions are supported.

  Default Value : `"#{application.name} (#{application.id})\nstatus changed from #{from.status} to #{to.status}\n\n#{application.healthUrl}"`



```
threes-org (3e68de5801fd) is OFFLINE

Instance 3e68de5801fd changed status from UP to OFFLINE

Status Details

exception
	io.netty.channel.AbstractChannel$AnnotatedConnectException
message
	Connection refused: no further information: localhost/127.0.0.1:9100
Registration

Service Url		http://localhost:9100/
Health Url		http://localhost:9100/actuator/health
Management Url	http://localhost:9100/actuator
```



