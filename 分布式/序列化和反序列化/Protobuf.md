# Protobuf

官方地址：<https://developers.google.cn/protocol-buffers>

Github：<https://github.com/protocolbuffers/protobuf>

## 一、简介



## 二、准备

工欲善其事，必先利其器。

### 2.1、安装Protobuf

下载Protobuf：<https://github.com/protocolbuffers/protobuf/releases>

截止2020-6-24，最新版本3.12.3。

格式为`protobuf-[language]-[version].[tar.gz|zip]`是源码包

格式为`protoc-[version]-[machine].zip`是安装包

##### 2.1.1、Windows安装



##### 2.1.2、Linux安装

1. 下载对应环境的Linux环境安装包

   ```shell
   $ wget https://github.com/protocolbuffers/protobuf/releases/download/v3.12.3/protoc-3.12.3-linux-x86_64.zip
   ```

2. 解压

   ```shell
   $ tar -zxvf protoc-3.12.3-linux-x86_64.zip
   ```

3. 解压之后目录结构如下如下：

   ```
   
   ```

4. 配置环境变量

   ```shell
   $ vim /etc/profile
   # 添加配置
   export PROTOC=/opt/protoc/bin
   # 保存退出:wq
   $ source /etc/profile
   ```

安装完成之后执行如下命令测试是否安装成功

```shell
$ protoc --version
libprotoc 3.12.3
```

### 2.2、安装IDEA插件

#### 2.2.1、protobuf-jetbrains-plugin

官方地址：<https://plugins.jetbrains.com/plugin/8277-protobuf-support>

Github：https://github.com/protostuff/protobuf-jetbrains-plugin

##### 1. IDEA插件安装

Intellij IDEA：`File → Settings → Plugin`，搜索插件`Protobuf Support`安装即可。

##### 2. 安装包安装

关于IDEA如何使用安装包安装插件，这里不过多描述。

截止2020-6-24最新版本0.13.0。下载地址如下：

下载最新版本：https://github.com/protostuff/protobuf-jetbrains-plugin/releases/download/v0.13.0/protobuf-jetbrains-plugin-0.13.0.zip

> 请根据其IDEA的版本下载对应的插件。

#### 2.2.2、GenProtobuf

官方地址：<https://plugins.jetbrains.com/plugin/11423-genprotobuf>

下载地址：https://plugins.jetbrains.com/files/11423/85146/genProtobuf.zip?updateId=85146&pluginId=11423&family=INTELLIJ

```
generate code from .proto files (they must have a .proto extension). GenProtobuf adds two right click menu entries "quick gen protobuf here" and "quick gen protobuf rules" to the project area and "Configure GenProtobuf" and "Generate all Protobufs" to the tools menu. quick gen protobuf here will quickly generate code for selected protobuf files in a single selected langauge as configured under "Configure GenProtobuf" in the tools menu placing the output in the same directory as the selected protobuf files. quick gen protobuf rules will generate code for selected protobuf files according to the rules set with "Configure GenProtobuf" in the tools menu and place the output in the configured output location. Finally you can use Generate all Protobufs under the tools menu and it will apply the rules set in Configure Genprotobuf (again under the tools menu,) to all the protobuf files it finds in your project. As stated in the descriptions above you can configure this plugin from the Configure GenProtobuf menu that gets added to the tools menu. You are able to set what programming language to use for the quick gen options and toggle different languages to be generated for protobuf files as well as different directories for the various language outputs. Whenever you generate code from a .proto file a tool window is created to display the resulting textual output.
```



## 三、快速开始

### 3.1、添加依赖

```xml
<dependency>
  <groupId>com.google.protobuf</groupId>
  <artifactId>protobuf-java</artifactId>
  <version>3.11.0</version>
</dependency>
```



### 3.2、编写proto脚本

### 3.3、转换成java类型







Maven插件：protobuf-maven-plugin