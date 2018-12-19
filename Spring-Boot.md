# Spring Boot

## [IV. Spring Boot features](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features)

### [23. SpringApplication](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-spring-application)

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

#### [23.1. Startup Failure](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-startup-failure)

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

#### [23.2. Customizing the Banner](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-banner)

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

#### [23.3. Customizing SpringApplication](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-customizing-spring-application)

如果默认的SpringApplication不符合你的口味，你可以创建一个本地实例并对它进行自定义。例如，想要关闭banner你可以这样写：

```java
public static void main(String[] args) {
	SpringApplication app = new SpringApplication(MySpringConfiguration.class);
	app.setBannerMode(Banner.Mode.OFF);
	app.run(args);
}
```

> ![](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/images/note.png) 传递给`SpringApplication`的构造器参数将作为spring beans的配置源，多数情况下，它们是一些`@Configuration`类的引用，但也可能是XML配置或要扫描包的引用。

你也可以使用`application.properties`文件来配置SpringApplication，具体参考24.Externalized

配置，访问[`SpringApplication` Javadoc](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/SpringApplication.html)可获取完整的配置选项列表。

#### [23.4. Fluent Builder API](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-fluent-builder-api)

如果需要创建一个分层的`ApplicationContext`（多个具有父子关系的上下文），或只是喜欢使用流式（fluent）构建API，那你可以使用`SpringApplicationBuilder`。`SpringApplicationBuilder`允许你以链式方式调用多个方法，包括`parent`和`child`方法，这个就可以创建多层次结构。例如：

```java
new SpringApplicationBuilder()
		.sources(Parent.class)
		.child(Application.class)
		.bannerMode(Banner.Mode.OFF)
		.run(args);
```

> ![](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/images/note.png) 创建`ApplicationContext`层次时有些限制，比如Web组件必须包含在子上下文中，并且父上下文和子上下文使用相同的`Environment`，具体参考[`SpringApplicationBuilder` Javadoc](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/builder/SpringApplicationBuilder.html)。

#### [23.5. Application Events and Listeners](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-application-events-and-listeners)？？？

除了常见的Spring框架事件，比如`ContextRefreshedEvent`，`SpringApplication`也会发送其他的application事件。

> ![](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/images/note.png) 有些事件实际上是在`ApplicationContext`创建前触发的，所以你不能在那些事件（处理类）中通过`@Bean`注册监听器，只能通过`SpringApplication.addListeners(...)`或`SpringApplicationBuilder.listeners(...)`方法注册。如果想让监听器自动注册，而不关心应用的创建方式，你可以在工程中添加一个`META/spring.factories`文件，并使用`org.springframework.context.ApplicationListener`作为key指向那些监听器，如下：
>
> ```properties
> org.springframework.context.ApplicationListener=\
> com.xlp.example.listener.HelloWorldApplicationListener
> ```

应用运行时，事件会以下面的次序发送：

1. `ApplicationStartingEvent`：
2. `ApplicationEnvironmentPreparedEvent`：在Environment将被用于已知的上下文，但在上下文被创建前
3. `ApplicationPreparedEvent`：在refresh开始前，但在bean定义已被加载后。
4. `ApplicationStartedEvent`：在运行开始，但除了监听器注册和初始化以外的任何处理之前。
5. `ApplicationReadyEvent`：在refresh之后，相关的回调处理完，会发此事件，表示应用准备好接收请求了。
6. `ApplicationFailedEvent`：启动过程中如果出现异常。

> ![](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/images/note.png) 通过不需要使用`application`事件，但知道他们的存在是有用的（在某些场合可能会使用到），比如，在Spring Boot内部会使用事件处理各种任务。

#### [23.6. Web Environment](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-web-environment)

`SpringApplication`尝试以`ApplicationContext`的名义创建正确的类型。用于确定它的算法`WebEnvironmentType`非常简单：

- 如果存在Spring MVC，使用`AnnotationConfigServletWebServerApplicationContext`。
- 如果Spring MVC不存在并且Spring WebFlux存在，使用`AnnotationConfigReactiveWebServerApplicationContext`。
- 否则，使用`AnnotationConfigApplicationContext`。

这意味着如果您的`WebClient`在同一应用程序中使用Spring MVC和Spring WebFlux中的新功能，则默认情况下会使用Spring MVC。您可以通过调用`setWebApplicationType(WebApplicationType)`轻松地覆盖它。

也可以完全控制`ApplicationContext`使用的类型：`setApplicationContextClass(…)`。

> ![](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/images/note.png) 在Junit测试中，使用`ApplicationContext`经常需要调用`setWebApplicationType(WebApplicationType.NONE)`。

#### [23.7. Accessing Application Arguments](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-application-arguments)

如果需要获取传递给`SpringApplication.run(...)`的应用参数，你可以注入一个`org.springframework.boot.ApplicationArguments`类型的bean。`ApplicationArguments`接口即提供对原始`String[]`参数的访问，也提供对解析成`option`和`non-option`参数的访问：

```java
import org.springframework.boot.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.stereotype.*;
@Component
public class MyBean {
	@Autowired
	public MyBean(ApplicationArguments args) {
		boolean debug = args.containsOption("debug");
		List<String> files = args.getNonOptionArgs();
		// if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
	}
}
```

![](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/images/note.png) Spring Boot也会注册一个包含Spring Environment属性的`CommandLinePropertySource`，这就允许你使用`@Value`注解注入单个的应用参数。

#### [23.8. Using the ApplicationRunner or CommandLineRunner](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-command-line-runner)

如果需要在SpringApplication启动后执行一些特殊的代码，你可以实现`ApplicationRunner`或`CommandLineRunner`接口，这两个接口工作方式相同，都只提供单一的`run`方法，该方法仅在`SpringApplication.run(...)`完成之前调用。

`CommandLineRunner`接口能够访问string数组类型的应用参数，而`ApplicationRunner`使用的是上面描述过的`ApplicationArguments`接口：

```java
@Component
public class MyBean implements CommandLineRunner {
	public void run(String... args) {
		// Do something...
	}
}
```

如果某些定义的`CommandLineRunner`或`ApplicationRunner`beans需要以特定的顺序调用，你可以实现`org.springframework.core.Ordered`接口或使用`org.springframework.core.annotation.Order`注解。

#### [23.9. Application Exit](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-application-exit)

每个`SpringApplication`向JVM注册一个关闭钩子，以确保`ApplicationContext`在退出时优雅地关闭。可以使用所有标准的Spring生命周期回调(例如`DisposableBean`接口或`@PreDestroy`注释)。另外，如果希望在调用`SpringApplication.exit()`时返回特定的退出代码，则可以实现`org.springframework.boot.ExitCodeGenerator`接口。然后可以将这个退出代码传递给`System.exit()`，将其作为状态代码返回，如下面的示例所示:

```java
@SpringBootApplication
public class ExitCodeApplication {

	@Bean
	public ExitCodeGenerator exitCodeGenerator() {
		return () -> 42;
	}

	public static void main(String[] args) {
		System.exit(SpringApplication
				.exit(SpringApplication.run(ExitCodeApplication.class, args)));
	}

}
```

此外，`ExitCodeGenerator`接口可以由异常实现。当遇到这种异常时，Spring Boot将返回实现的`getExitCode()`方法提供的退出代码。

#### [23.10. Admin Features](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-application-admin)

通过指定`spring.application.admin.enabled`属性可以为应用程序启用与管理相关的功能。这暴露了`SpringApplicationAdminMXBean`平台上的`MBeanServer`。您可以使用此功能远程管理您的Spring Boot应用程序。此功能对于任何服务包装器实现也可能有用。

> ![](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/images/tip.png) 如果您想知道应用程序在那个HTTP端口上运行，请使用键来获取属性`local.server.port`。

> ![](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/images/caution.png) 启用此功能时要小心，因为MBean公开了关闭应用程序的方法。

## [24. Externalized Configuration](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config)

Spring Boot允许将配置外部化（externalized），这样你就能够在不同的环境下使用相同的代码。你可以使用properties文件，YAML文件，环境变量和命令行参数来外部化配置。使用`@Value`注解，可以直接将属性值注入到beans中，然后通过Spring的`Environment`抽象或通过`@ConfigurationProperties`绑定到结构化对象来访问。

Spring Boot设计了一个非常特别的PropertySource顺序，以允许对属性值进行合理的覆盖，属性会以如下的顺序进行设置值：

1. home目录下的devtools全局设置属性（`~/.spring-boot-devtools.properties`，如果devtools激活）。
2. 测试用例上的`@TestPropertySource`注解。
3. 测试用例上的`@SpringBootTest#properties`注解。
4. 命令行参数。
5. 来自`SPRING_APPLICATION_JSON`的属性（环境变量或系统属性中内嵌的内联JSON）。
6. `ServletConfig`初始化参数。
7. `ServletContext`初始化参数。
8. 来自于`java:comp/env`的JDNI属性。
9. Java系统属性（`System.getProperties()`）。
10. 操作系统环境变量
11. `RandomValuePropertySource`，只包含`random.*`中的属性。
12. 没有打进jar包的`Profile-specific`应用属性（`application-{profile}.properties`和`YAML`变量）。
13. 打进jar包的`Profile-specific`应用属性（`application-{profile}.properties`和`YAML`变量）。
14. 没有打进jar包的应用配置（`application.properties`和`YAML`变量）。
15. 打进jar包的应用配置（`application.properties`和`YAML`变量）。
16. `@Configuration`类上的`@PropertySource`注解。
17. 默认属性（使用`SpringApplication.setDefaultProperties`指定）。

下面是具体的示例，假设你开发一个使用name属性的`@component`：

```java
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...
}
```

你可以将一个`application.properties`放到应用的classpath下，为`name`提供一个合适的默认属性值。当在新的环境中运行时，可以在jar包外提供一个`application.properties`覆盖`name`属性。对于一次性的测试，你可以使用特定的命令行开关启动应用（比如，`java -jar app.jar --name="Spring"`）。

> ![](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/images/tip.png)`SPRING_APPLICATION_JSON`属性可以通过命令行的环境变量设置，例如，在一个UN*X shell中可以这样：
>
> ```shell
> $ SPRING_APPLICATION_JSON='{"foo":{"bar":"spam"}}' java -jar myapp.jar
> ```
>
> 本示例中，如果是Spring Environment，你可以以`foo.bar=spam`结尾；如果在一个系统变量中，可以提供作为`spring.application.json`的JSON字符串：
>
> ```shell
> $ java -Dspring.application.json='{"foo":"bar"}' -jar myapp.jar
> ```
>
> 或命令行参数：
>
> ```shell
> $ java -jar myapp.jar --spring.application.json='{"foo":"bar"}'
> ```
>
> 或作为一个JNDI变量`java:comp/env/spring.application.json`。

### [24.1. Configuring Random Values](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-random-values)



### [24.2. Accessing Command Line Properties](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-command-line-args)

默认情况下，`SpringApplication`会将所有命令行配置参数（以`--`开头，比如`--server.port=9000`）转化成一个`property`，并将其添加到`Spring Environment`中。正如以上章节提过的，命令行属性总是优于其他属性源。

如果不想将命令行属性添加到`Environment`，你可以使用`springApplication.setAddCommandLineProperties(false);`来禁用它们。

### [24.3. Application Property Files](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-application-property-files)



[24.4. Profile-specific Properties](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-profile-specific-properties)

[24.5. Placeholders in Properties](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-placeholders-in-properties)

[24.6. Encrypting Properties](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-encrypting-properties)

[24.7. Using YAML Instead of Properties](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-yaml)

- [24.7.1. Loading YAML](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-loading-yaml)
- [24.7.2. Exposing YAML as Properties in the Spring Environment](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-exposing-yaml-to-spring)
- [24.7.3. Multi-profile YAML Documents](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-multi-profile-yaml)
- [24.7.4. YAML Shortcomings](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-yaml-shortcomings)

[24.8. Type-safe Configuration Properties](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-typesafe-configuration-properties)

- [24.8.1. Third-party Configuration](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-3rd-party-configuration)

- [24.8.2. Relaxed Binding](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-relaxed-binding)

- [24.8.3. Merging Complex Types](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-complex-type-merge)

- [24.8.4. Properties Conversion](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-conversion)

  [Converting durations](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-conversion-duration)[Converting Data Sizes](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-conversion-datasize)

- [24.8.5. @ConfigurationProperties Validation](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-validation)

- [24.8.6. @ConfigurationProperties vs. @Value](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-external-config-vs-value)



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

