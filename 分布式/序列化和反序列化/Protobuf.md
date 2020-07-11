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

1. 下载Windows环境安装包

   https://github.com/protocolbuffers/protobuf/releases/download/v3.12.3/protoc-3.12.3-win64.zip
   
2. 解压

3. 解压之后目录结构如下

   ```
   ├── bin
   │   └── protoc.exe
   ├── include
   │   └── google
   │       └── protobuf
   │           ├── any.proto
   │           ├── api.proto
   │           ├── compiler
   │           │   └── plugin.proto
   │           ├── descriptor.proto
   │           ├── duration.proto
   │           ├── empty.proto
   │           ├── field_mask.proto
   │           ├── source_context.proto
   │           ├── struct.proto
   │           ├── timestamp.proto
   │           ├── type.proto
   │           └── wrappers.proto
   ├── protoc-3.12.3-win64.zip
   └── readme.txt
   ```

4. 配置环境变量

   配置指定protoc.exe文件的环境变量

   ```shell
   PROTOC=/path/bin
   ```

##### 2.1.2、Linux安装

1. 下载对应环境的Linux环境安装包

   ```shell
   $ wget https://github.com/protocolbuffers/protobuf/releases/download/v3.12.3/protoc-3.12.3-linux-x86_64.zip
   ```

2. 解压

   ```shell
   $ unzip protoc-3.12.3-linux-x86_64.zip
   ```

3. 解压之后目录结构如下：

   ```
   ├── bin
   │   └── protoc
   ├── include
   │   └── google
   │       └── protobuf
   │           ├── any.proto
   │           ├── api.proto
   │           ├── compiler
   │           │   └── plugin.proto
   │           ├── descriptor.proto
   │           ├── duration.proto
   │           ├── empty.proto
   │           ├── field_mask.proto
   │           ├── source_context.proto
   │           ├── struct.proto
   │           ├── timestamp.proto
   │           ├── type.proto
   │           └── wrappers.proto
   └── readme.txt
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

下载地址：https://github.com/protostuff/protobuf-jetbrains-plugin/releases

Google Protobuf support for JetBrains products.  

Features:

- Full Proto3 support.
- Custom include path for proto files.
- Usage search for messages, enums and fields (for standard and custom options).
- Syntax validation for proto2/proto3. Checks for reserved/duplicated field tags and names. Highlight unresolved reference errors.
- Fonts & Colors configuration.
- Structure View.
- Code formatting.
- Navigation to message, enum or service by name (Ctrl+N)
- Rename refactoring (files, messages, enums and fields).
- Spell checking.

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

```protobuf
syntax = "proto3";

option java_package="com.demo.protobuf";

message User {
    int32 id = 2;
    string name = 1;
    string email = 3;
}
```

### 3.3、转换成java类型

```shell
$ protoc -I src/main/resources/proto/ --java_out=src/main/java/ user.proto
```

>- `-I`：指定.proto文件所在路径。
>- `--java_out`：编译成java文件时，文件除输出路径。
>- `user.proto`：执行需要编译的.proto文件。

### 3.4、序列化和反序列化

```java
public static void main( String[] args ) throws InvalidProtocolBufferException {
    //User构造器
    UserOuterClass.User.Builder userBuilder = UserOuterClass.User.newBuilder();
    //User赋值
    userBuilder.setId(1);
    userBuilder.setName("xlp");
    userBuilder.setEmail("xlp@qq.com");
    //生成User对象
    UserOuterClass.User userClass = userBuilder.build();
    //序列化
    byte[] bytes = userClass.toByteArray();
    //反序列化
    UserOuterClass.User user = UserOuterClass.User.parseFrom(bytes);
    System.out.println(user);
}
```





## Maven插件：protobuf-maven-plugin



```xml
<dependency>
    <groupId>org.xolstice.maven.plugins</groupId>
    <artifactId>protobuf-maven-plugin</artifactId>
    <version>0.6.1</version>
</dependency>
```





## 附录

```shell
$ protoc -h
Usage: protoc [OPTION] PROTO_FILES
Parse PROTO_FILES and generate output based on the options given:
  -IPATH, --proto_path=PATH   Specify the directory in which to search for
                              imports.  May be specified multiple times;
                              directories will be searched in order.  If not
                              given, the current working directory is used.
                              If not found in any of the these directories,
                              the --descriptor_set_in descriptors will be
                              checked for required proto file.
  --version                   Show version info and exit.
  -h, --help                  Show this text and exit.
  --encode=MESSAGE_TYPE       Read a text-format message of the given type
                              from standard input and write it in binary
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --decode=MESSAGE_TYPE       Read a binary message of the given type from
                              standard input and write it in text format
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --decode_raw                Read an arbitrary protocol message from
                              standard input and write the raw tag/value
                              pairs in text format to standard output.  No
                              PROTO_FILES should be given when using this
                              flag.
  --descriptor_set_in=FILES   Specifies a delimited list of FILES
                              each containing a FileDescriptorSet (a
                              protocol buffer defined in descriptor.proto).
                              The FileDescriptor for each of the PROTO_FILES
                              provided will be loaded from these
                              FileDescriptorSets. If a FileDescriptor
                              appears multiple times, the first occurrence
                              will be used.
  -oFILE,                     Writes a FileDescriptorSet (a protocol buffer,
    --descriptor_set_out=FILE defined in descriptor.proto) containing all of
                              the input files to FILE.
  --include_imports           When using --descriptor_set_out, also include
                              all dependencies of the input files in the
                              set, so that the set is self-contained.
  --include_source_info       When using --descriptor_set_out, do not strip
                              SourceCodeInfo from the FileDescriptorProto.
                              This results in vastly larger descriptors that
                              include information about the original
                              location of each decl in the source file as
                              well as surrounding comments.
  --dependency_out=FILE       Write a dependency output file in the format
                              expected by make. This writes the transitive
                              set of input file paths to FILE
  --error_format=FORMAT       Set the format in which to print errors.
                              FORMAT may be 'gcc' (the default) or 'msvs'
                              (Microsoft Visual Studio format).
  --print_free_field_numbers  Print the free field numbers of the messages
                              defined in the given proto files. Groups share
                              the same field number space with the parent
                              message. Extension ranges are counted as
                              occupied fields numbers.

  --plugin=EXECUTABLE         Specifies a plugin executable to use.
                              Normally, protoc searches the PATH for
                              plugins, but you may specify additional
                              executables not in the path using this flag.
                              Additionally, EXECUTABLE may be of the form
                              NAME=PATH, in which case the given plugin name
                              is mapped to the given executable even if
                              the executable's own name differs.
  --cpp_out=OUT_DIR           Generate C++ header and source.
  --csharp_out=OUT_DIR        Generate C# source file.
  --java_out=OUT_DIR          Generate Java source file.
  --js_out=OUT_DIR            Generate JavaScript source.
  --objc_out=OUT_DIR          Generate Objective C header and source.
  --php_out=OUT_DIR           Generate PHP source file.
  --python_out=OUT_DIR        Generate Python source file.
  --ruby_out=OUT_DIR          Generate Ruby source file.
  @<filename>                 Read options and filenames from file. If a
                              relative file path is specified, the file
                              will be searched in the working directory.
                              The --proto_path option will not affect how
                              this argument file is searched. Content of
                              the file will be expanded in the position of
                              @<filename> as in the argument list. Note
                              that shell expansion is not applied to the
                              content of the file (i.e., you cannot use
                              quotes, wildcards, escapes, commands, etc.).
                              Each line corresponds to a single argument,
                              even if it contains spaces.
```

