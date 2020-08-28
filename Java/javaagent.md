# java agent

## *What is it?*

Java编程语言提供了agent服务，允许来检测运行在JVM上的程序。

Javaagent是JDK 1.5引入的特性，其主要作用是在class被加载之前对其拦截，如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190321234650938.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMyNDk3MzYx,size_16,color_FFFFFF,t_70)

## *What can do?*





Agent作为JAR文件部署。JAR文件清单中的一个属性指定了agent类，它将被加载以启动agent。Agent可以通过以下几种方式启动：

1. 支持命令行接口实现，可以通过在命令行上指定`-javaagent`选项来启动agent。
2. 支持在虚拟机启动一段时间之后再启动agent。
3. Agent可以与应用程序打包在可执行的JAR文件中。





## *Starting an Agent from the Command-Line Interface*

通过向命令行添加如下选项启动agent：

```shel
-javaagent:<jarpath>[=<options>]
```

Agent JAR文件的主清单中必须包含`Premain-Class`属性。这个属性的值是*agent class*的全类名。agent class必须实现`public static premain`方法，原理类似于主应用程序的入口点（main函数）。初始化Java虚拟机之后，`premain`方法将被调用。然后才调用应用程序的`main`方法。`premain`方法必须返回，才能继续启动。

`premain`方法有两个可能的签名。JVM首先尝试在agent类上调用以下方法：

```java
public static void premain(String agentArgs, Instrumentation inst)
```

如果Agent类没有实现这个方法，将尝试调用：

```java
public static void premain(String agentArgs)
```

**其他特性：**

- Agent类还可以包含`agentmain`方法（见下一节），但当使用命令行的方式启动Agent时，`agentmain`方法不会被调用。

- 每个Agent都可以通过`agentArgs`传递其Agent选项，Agent选项以单个字符串的方式传递，对其的解析有Agent本身执行。

- Agent如果存在以下两种情况，JVM将中止：

    1. Agent无法启动（例如：Agent类无法加载，或者Agent类没有`premain`方法）。
    2. `premain`方法抛出未捕获的异常。

- 在同一命令行上可以多次使用`-javaagent`选项，从而启动多个Agent。

    ```shell
    $ java -javaagent=... -javaagent=... -javaagent=... -jar ...
    ```

    `premain`方法将按照在命令行中声明的顺序执行。多个Agent可以使用相同的`<jarpath>`。

- 对于Agent的`premain`方法没有什么限制，任何`main`函数可以做的事情，包括创建线程等，`premain`方法都可以做。

### Operation Example

1. 编写Agent类

   ```java
   
   ```

2. 自定义JAR文件清单文件`MANIFEST.MF`

   ```
   
   ```

   

   - 手动导入`MANIFEST.MF`清单文件
   - Maven插件生成`MANIFEST.MF`清单文件

3. 打包为JAR文件

   ```shell
   $ mvn clean -Dmaven.test.skip=true package
   ```

4. 新启项目，添加命令行参数`-javaagent`指定Agent JAR文件

   

5. 启动main函数测试

## Starting an Agent After VM Startup

Java Agent支持在虚拟机启动之后启动，此时，应用程序已经启动，Main函数已经被调用。可以采用以下方法实现：

1. 