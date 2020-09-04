# IDEA Plugins



## jclasslib

jclasslib字节码查看器是一个工具，它可以可视化已编译的Java类文件和所包含的字节码的所有方面。此外，它还包含一个库，允许开发人员读写Java类文件和字节码。

Githup：https://github.com/ingokegel/jclasslib

IDEA Plugin：https://plugins.jetbrains.com/plugin/9248-jclasslib-bytecode-viewer

IDEA插件通过视图菜单`view`—>`Show Bytecode With jclasslib`打开：

![](/images/1597114009267.png)

## Docker

提供与Docker的集成

- 下载并构建Docker镜像
- 从拉取的镜像或直接从Dockerfile创建和运行Docker容器
- 使用专用的Docker运行配置
- 使用Docker Compose运行多容器应用程序

IDEA Plugin：https://plugins.jetbrains.com/plugin/7724-docker

## Kubernetes

IDEA上支持编辑Kubernetes资源文件的插件。

Kubernetes插件将查找文件中是否存在*apiVersion*和*kind*字段，如果存在这些字段，则会将此类文件视为*Kubernetes资源*文件。

相关使用资源：https://blog.csdn.net/ccc7574/article/details/85679015

IDEA Plugin：https://plugins.jetbrains.com/plugin/10485-kubernetes



## VisualVM launcher

查看内存等实时动态信息



IDEA Plugin：https://plugins.jetbrains.com/plugin/7115-visualvm-launcher



## JRebel

参见Tools → JRebel

## Protobuf Support



## Mybatis Log Plugin



## Alibaba Java Coding Guidelines





## nginx support





## Alibaba Cloud Toolkit

一键部署插件





- lombok

- mybatisx
   同类的有mybatis tools.但是被我放弃了;因为mybatisx的小鸟图标很可爱,并且可以自动生产sql的模块,mybatis tools中并没有

- mybatis log plugin
   能够自我调试的时候,快速将日志中的sql拼成可以直接使用的sql

- translate
   听说大佬们在学习途中都是逐渐从翻译到一眼能看懂, 我想朝这个方向进步一下

- translation
   同样用于翻译; 后期我感觉这个更好用一些

- jrebel
   热部署工具,[破解推荐](https://links.jianshu.com/go?to=%5Bhttps%3A%2F%2Fblog.csdn.net%2Fqierkang%2Farticle%2Fdetails%2F95095954%5D(https%3A%2F%2Fblog.csdn.net%2Fqierkang%2Farticle%2Fdetails%2F95095954))
   带有一个特别好的功能:xrebel,这个是用于查看请求响应的链路和耗时,特别推荐

  

- Key Promoter X
   开启快捷键之路; 不用快捷键就被提醒

#### 特殊需求插件

- arthas idea
   只是一个快捷生成指令的插件: arthas使用时,经常需要指令后需要跟类全路径+方法名,用插件就会自动生成指令