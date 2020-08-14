# JRebel

Official：<https://www.jrebel.com/>

Official Document：<https://manuals.jrebel.com/jrebel/>

## Introduction

JRebel是一套JavaEE开发工具。JRebel允许开发团队在有限的时间内完成更多的任务修正更多的问题，发布更高质量的软件产品。 JRebel是收费软件，用户可以在JRebel官方站点下载30天的评估版本。

## What Is JRebel?

JRebel is a JVM plugin that fast-tracks the development of Java applications. It allows developers to view code changes in real time while preserving application state.

JRebel integrates with 100+ leading Java frameworks, as well as application servers, IDEs and build environments.

## What Can JRebel Do?

JRebel empowers developers to create better applications, faster.

- Skip Rebuilds and Redeploys（跳过重新构建和重新部署）

  Rebuilds and redeploy times add up fast. Stop waiting and keep coding.

- Real-Time Change Visibility（变化实时性可见）

  View the results of iterative code and resource changes to a Java application in real time.

- Maintain Application State（维护应用程序状态）

  Avoid the time spent reproducing the pre-change application state after a redeploy.

- Finish Your Sprint Early（提前完成Sprint）

  JRebel increases team velocity up to 40% (backed by surveys and[ case studies](https://www.jrebel.com/customers)).

## Quick start

### IntelliJ IDEA

#### Installation

**Installing the JRebel pligun**

1. 访问**File→Settings→Plugins**。
2. 选择**Marketplace**。
3. 搜索**JRebel and XRebel for Intellij**，点击**Install plugin**。

Didn't work?

如果不能通过IDEA自动安装插件，可以到此地址<https://plugins.jetbrains.com/idea/plugin/4441-jrebel-for-intellij>下载插件安装。

如果安装成功，IDE将重新启动。重启后，JRebel将通过一个通知通知您。

#### Activation

JRebel是付费插件，需要激活。

激活方式：

1. 下载激活工具：<https://github.com/ilanyu/ReverseProxy/releases/tag/v1.4>

2. 生成GUID：<https://www.guidgen.com/>

3. 配置激活

   JRebel是收费软件。激活方式如下：

   1. 启动激活工具

      直接运行**ReverseProxy_{xxx}**文件即可。

   2. 填写激活URL和邮箱

      激活URL以`http://127.0.0.1:8888`为前缀，后接`GUID`，格式为`http://127.0.0.1:8888/{GUID}`。

      URL为激活工具启动的服务，GUID为第二步生成的`GUID`。

      邮箱只要符合邮箱格式即可。

   3. 勾选**I agree with the terms & conditions of the License Agreement**。

   4. 点击**Activate JRebel**激活。

   如下所示：

   ![img](/images/15972021363852.png)

   5. 激活成功

      ![img](/images/15972150002138.png)

4. 设置离线模式

   设置离线模式可以180天内不用再次激活，可随时重新点下**Renew offline Seat**刷新激活周期，180天后激活状态会重新刷新。

   访问**File→Settings→JRebel & XRebel**，点击**Work online**，之后直接界面如下：

   ![img](/images/15972023828364.png)

## Enable automatic build compilation

有如下四种设置方式：

1. 重新启动应用程序

2. 手动触发编译

   点击**Build > Build Project**或快捷键**Cirl+F9**。

3. IDEA自动编译

   1. 访问**File→Settings→Build,Execution,Deployment→Compiler**，勾选**Build project automatically**，

   2. 点击**Ctrl+Shift+A**或选择**Help→Find Action**，搜索**Registry**

      ![img](/images/15972087734291.png)

      勾选**compiler.automake.allow.when.app.running**

      ![img](/images/15972092101919.png)

4. JRebel启动自动编译

   访问**Help > JRebel > Configuration > Advanced > Miscellaneous**，勾选**Enable Intellij automatic compilation**。其本质就跟设置IDEA自动编译是一样的，它会自动触发IDEA自动编译的设置。

   ![img](/images/15972102853587.png)



## Enable project and test heat deployment

启动项目，控制台会输出如下内容：

```
XRebel: Starting logging to file: C:\Users\{home}\.xrebel\xrebel.log
2020-08-12 13:43:56 JRebel:  Starting logging to file: C:\Users\{home}\.jrebel\jrebel.log
2020-08-12 13:43:56 JRebel:  
2020-08-12 13:43:56 JRebel:  ############################################################
2020-08-12 13:43:56 JRebel:  
2020-08-12 13:43:56 JRebel:  JRebel Agent 2020.2.4 (202007241243)
2020-08-12 13:43:56 JRebel:  (c) Copyright 2007-2020 Perforce Software, Inc.
2020-08-12 13:43:56 JRebel:  
2020-08-12 13:43:56 JRebel:  Over the last 1 days JRebel prevented
2020-08-12 13:43:56 JRebel:  at least 9 redeploys/restarts saving you about 0.1 hours.
2020-08-12 13:43:56 JRebel:  
2020-08-12 13:43:56 JRebel:  License acquired from License Server: http://127.0.0.1:8888
2020-08-12 13:43:56 JRebel:  
2020-08-12 13:43:56 JRebel:  Licensed to ilanyu.
2020-08-12 13:43:56 JRebel:  
2020-08-12 13:43:56 JRebel:  You are using an offline license.
2020-08-12 13:43:56 JRebel:  
2020-08-12 13:43:56 JRebel:  
2020-08-12 13:43:56 JRebel:  ############################################################
2020-08-12 13:43:56 JRebel:  
XRebel: 
XRebel: ################################################################
XRebel: 
XRebel:  XRebel 2020.2.0 (202006020843)
XRebel:  (c) Copyright ZeroTurnaround AS, Estonia, Tallinn.
XRebel: 
XRebel:  For questions and support, contact xrebel@zeroturnaround.com
XRebel: 
XRebel: ################################################################
XRebel: 
```

修改代码，并重新编译（手动或自动），会触发热部署，控制台输出如下内容：

```
2020-08-12 13:46:10 JRebel: Reloading class '{package.class}'.
2020-08-12 13:46:10 JRebel: Reconfiguring bean '{beanName}' [{package.class}]
```


## 搭建自定义的激活服务器

**ReverseProxy_{xxx}**文件本质上还是启动一个服务代理转发到真正的激活服务地址。
但是在没有网络的情况下，这个文件就不好使了。这个时候我们可以搭建自己的激活服务器。

Gitee提供了用于搭建激活服务器的源码，其地址为：[](https://gitee.com/gsls200808/JrebelLicenseServerforJava)

该项目已经详细声明其使用方式：

- Setup

	Run:
	
	```shell
	cd /path/to/project
	mvn compile 
	mvn exec:java -Dexec.mainClass="com.vvvtimes.server.MainServer" -Dexec.args="-p 8081"
	```
	
	Packing a runnable jar:
	
	```shell
	mvn package
	```
	
	then
	
	```shell
	java -jar JrebelBrainsLicenseServerforJava-1.0-SNAPSHOT.jar -p 8081
	```
	default port is 8081.
	
	Or use gradle
	
	```shell
	gradle shadowJar
	
	java -jar JrebelBrainsLicenseServerforJava-1.0-SNAPSHOT.jar -p 8081
	```
	
- Docker
	Build image
	
	```shell
	mvn package
	docker build -t jrebel-ls .
	```
	
	start container
	
	```shell
	docker run -d --name jrebel-ls --restart always -e PORT=9001 -p 9001:9001 jrebel-ls
	```
	
	default port is `8081`,you can modify it.
	
- Support

	Jrebel and XRebel

	JRebel for Android

	JetBrains Products







