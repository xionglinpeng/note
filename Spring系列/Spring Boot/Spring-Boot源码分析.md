# Spring Boot源码分析

## 起步依赖starter



## 自动配置autoconfigure



SpringApplication



由不可枚举的Spring PropertySource或受限制的EnumerablePropertySource实现支持的ConfigurationPropertySource(例如受安全限制的systemEnvironment源代码)。PropertySource在PropertyMapper的帮助下进行了调整，后者为各个属性提供了映射规则。每个getConfigurationProperty调用都试图将ConfigurationPropertyName映射到一个或多个基于字符串的名称。这允许对格式良好的属性源进行快速的属性解析。

如果可能的话，SpringIterableConfigurationPropertySource将优先用于此实现，因为它支持完全“relaxed”样式的解析。





详细说明了PropertySource项和ConfigurationPropertySource项之间的映射。



## 外部化配置



## actuator

## configuration-processor

## dev-tool













































# Spring Boot源码分析





```java
/**
	 * Create a new {@link SpringApplication} instance. The application context will load
	 * beans from the specified primary sources (see {@link SpringApplication class-level}
	 * documentation for details. The instance can be customized before calling
	 * {@link #run(String...)}.
	 * @param primarySources the primary bean sources
	 * @see #run(Class, String[])
	 * @see #SpringApplication(ResourceLoader, Class...)
	 * @see #setSources(Set)
	 */
public SpringApplication(Class<?>... primarySources) {
    this(null, primarySources);
}

/**
	 * Create a new {@link SpringApplication} instance. The application context will load
	 * beans from the specified primary sources (see {@link SpringApplication class-level}
	 * documentation for details. The instance can be customized before calling
	 * {@link #run(String...)}.
	 * @param resourceLoader the resource loader to use
	 * @param primarySources the primary bean sources
	 * @see #run(Class, String[])
	 * @see #setSources(Set)
	 */
@SuppressWarnings({ "unchecked", "rawtypes" })
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    //推断WEB应用类型
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    //
    setInitializers((Collection) getSpringFactoriesInstances(
        ApplicationContextInitializer.class));
    //设置监听器
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //推断引导类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```







```java
/**
 * 运行Spring应用程序，创建和刷新一个新的{@link ApplicationContext}。
 * @param args 应用程序参数(通常从Java main方法传递)
 * @return 正在运行的{@link ApplicationContext}
 */
public ConfigurableApplicationContext run(String... args) {
   //
   StopWatch stopWatch = new StopWatch();
   //
   stopWatch.start();
   ConfigurableApplicationContext context = null;
   //
   Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
   //
   configureHeadlessProperty();
   //获取SpringApplication运行监听器
   SpringApplicationRunListeners listeners = getRunListeners(args);
   //
   listeners.starting();
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      //准备环境 
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
      // 
      configureIgnoreBeanInfo(environment);
      //答应Banner
      Banner printedBanner = printBanner(environment);
      //创建应用上下文
      context = createApplicationContext();
      // 
      exceptionReporters = getSpringFactoriesInstances(
            SpringBootExceptionReporter.class,
            new Class[] { ConfigurableApplicationContext.class }, context);
      //准备上下文
      prepareContext(context, environment, listeners, applicationArguments,
            printedBanner);
      //刷新上下文
      refreshContext(context);
      // 
      afterRefresh(context, applicationArguments);
      // 
      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass)
               .logStarted(getApplicationLog(), stopWatch);
      }
      //ConfigurableApplicationContext已启动，此时Spring Bean已初始化完成
      listeners.started(context);
      //执行runner接口方法，用于在SpringApplication启动后执行一些特殊的代码
      callRunners(context, applicationArguments);
   }
   catch (Throwable ex) {
      //处理运行失败
      handleRunFailure(context, ex, exceptionReporters, listeners);
      throw new IllegalStateException(ex);
   }

   try {
      //Spring应用正在运行
      listeners.running(context);
   }
   catch (Throwable ex) {
      //处理运行失败 
      handleRunFailure(context, ex, exceptionReporters, null);
      throw new IllegalStateException(ex);
   }
   return context;
}
```





### 类型推断



