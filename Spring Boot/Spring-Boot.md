## [23. SpringApplication](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-spring-application)

SpringApplication类提供了一种快捷方式，用于从main()方法启动Spring应用。多数情况下，你只需要将该任务委托给`SpringApplication.run`静态方法：

```java
public static void main(String[] args) {
	SpringApplication.run(MySpringConfiguration.class, args);
}
```

当应用启动时，你应该会看到类似下面的东西：

```
 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v2.1.0.RELEASE

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatServletWebServerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```

默认情况下会显示INFO级别的日志信息，包括一些相关的启动详情，比如启动应用的用户等。

### [23.1. Startup Failure](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-startup-failure)

如果应用启动失败，注册的`FailureAnalyzers`就有机会提供一个特定的错误信息，及具体的解决该问题的动作。例如，如果在8080端口启动一个web应用，而该端口已被占用，那你应该可以看到类似如下的内容：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

> ![](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/images/note.png)Spring Boot提供很多的`FailureAnalyzer`实现，你自己实现也很容易。

如果没有可用于处理该异常的失败分析器（failure analyzers），你需要展示完整的auto-configuration报告以便更好的查看出问题的地方，因此你需要启用`org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener`的debug属性，或开启DEBUG日志级别。

例如，使用java -jar运行应用时，你可以通过如下命令启用debug属性：

```shell
java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### [23.2. Customizing the Banner](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-banner)

通过在classpath下添加一个`banner.txt`或设置`banner.location`来指定相应的文件可以改变启动过程中打印的banner。如果这个文件有特殊的编码，你可以使用`banner.encoding`设置它（默认为UTF-8）。除了文本文件，你也可以添加一个`banner.gif`，`banner.jpg`或`banner.png`图片，或设置`banner.image.location`属性。图片会转换为字符画（ASCII art）形式，并在所有文本banner上方显示。

在banner.txt中可以使用如下占位符：

| Variable                                                     | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `${application.title}`                                       | `MANIFEST.ME`中声明的应用title，例如`Implementation-Title:MuApp`会打印`MyApp`。 |
| `${application.version}`                                     | `MANIFEST.ME`中声明的应用版本号，例如`Implementation-Version:1.0`会1.0。 |
| `${application.formatted-version}`                           | `MANIFEST.ME`中声明的被格式化后的应用版本号（被括号包裹且与`v`作为前缀），用于显示。例如`(v1.0)`。 |
| `${spring-boot.version}`                                     | 当前Spring Boot的版本号，例如`2.1.0.RELEASE`。               |
| `${spring-boot.formatted-version}`                           | 当前Spring Boot被格式化后的版本号，（被括号包裹且与`v`作为前缀），用于显示。例如`(v2.1.0.RELEASE)`。 |
| `${Ansi.NAME}`,<br />`${AnsiColor.NAME}`,<br />`${AnsiBackground.NAME}`,<br />`${AnsiStyle.NAME}` | NAME代表一种AMSI编码，具体详情查看[`AnsiPropertySource`](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)。 |

> ![](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/images/note.png)如果想以编程的方式产生一个banner，可以使用`springApplication.setBanner(...)`方法，并实现`org.springframework.boot.Banner`接口的`printBanner()`方法。

你也可以使用`spring.main.banner-mode`属性决定将banner打印到何处， `System.out` (`console`)，配置的logger (`log`)，或都不输出(`off`)。默认是`console`。

打印的banner将注册成为一个名为`springBootBanner`的单例Bean。

> ![](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/images/note.png)YAML会将off映射为false，如果想在应用中禁用banner，你需要确保off添加了括号：
>
> ```yaml
> spring:
>   main:
>     banner-mode: "off"
> ```

### [23.3. Customizing SpringApplication](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-customizing-spring-application)

- [23.4. Fluent Builder API](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-fluent-builder-api)
- [23.5. Application Events and Listeners](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-application-events-and-listeners)
- [23.6. Web Environment](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-web-environment)
- [23.7. Accessing Application Arguments](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-application-arguments)
- [23.8. Using the ApplicationRunner or CommandLineRunner](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-command-line-runner)
- [23.9. Application Exit](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-application-exit)
- [23.10. Admin Features](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-application-admin)