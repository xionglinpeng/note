# 使用模块

## 3.1、第一个模块



### 3.1.1、剖析模块





```
.
└── src
    └── main
        └── helloworld
            ├── com
            │   └── javamodularity
            │       └── helloworld
            │           └── HelloWorld.java
            └── module-info.java

```





### 3.1.2、命名模块



### 3.1.3、编译



```shell
$ javac -d out/helloworld src/main/helloworld/com/javamodularity/helloworld/HelloWorld.java src/main/helloworld/module-info.java
```





```shell
.
└── out
    └── helloworld
        ├── com
        │   └── javamodularity
        │       └── helloworld
        │           └── HelloWorld.class
        └── module-info.class
```





### 3.1.4、打包



```shell
$ jar -cfev out/helloworld.jar com.javamodularity.helloworld.HelloWorld -C out/helloworld .
```



```
.
└── mods
    └── helloworld.jar
```



### 3.1.5、运行模块



```shell
$ java --module-path out --module helloworld/com.javamodularity.helloworld.HelloWorld
Hello Modular World!
$ java --module-path mods --module helloworld/com.javamodularity.helloworld.HelloWorld
Hello Modular World!
$ java --module-path mods --module helloworld
Hello Modular World!
```









### 3.1.6、模块路径



### 3.1.7、链接模块





```shell
$ jlink --module-path mods/:$JAVA_HOME/jmods --add-modules helloworld --launcher hello=helloworld --output helloworld-image
$ sh helloworld-image/bin/hello
Hello Modular World!
```









```
.
└── helloworld-image
    ├── bin
    │   ├── hello
    │   ├── java
    │   └── keytool
    ├── conf
    │   ├── net.properties
    │   └── security
    │       ├── java.policy
    │       ├── java.security
    │       └── policy
    │           ├── limited
    │           │   ├── default_local.policy
    │           │   ├── default_US_export.policy
    │           │   └── exempt_local.policy
    │           ├── README.txt
    │           └── unlimited
    │               ├── default_local.policy
    │               └── default_US_export.policy
    ├── include
    │   ├── classfile_constants.h
    │   ├── jni.h
    │   ├── jvmticmlr.h
    │   ├── jvmti.h
    │   └── linux
    │       └── jni_md.h
    ├── legal
    │   └── java.base
    │       ├── aes.md
    │       ├── asm.md
    │       ├── cldr.md
    │       ├── c-libutl.md
    │       ├── COPYRIGHT
    │       ├── icu.md
    │       ├── LICENSE
    │       ├── public_suffix.md
    │       └── unicode.md
    ├── lib
    │   ├── classlist
    │   ├── jexec
    │   ├── jli
    │   │   └── libjli.so
    │   ├── jrt-fs.jar
    │   ├── jvm.cfg
    │   ├── libjava.so
    │   ├── libjimage.so
    │   ├── libjsig.so
    │   ├── libnet.so
    │   ├── libnio.so
    │   ├── libverify.so
    │   ├── libzip.so
    │   ├── modules
    │   ├── security
    │   │   ├── blacklisted.certs
    │   │   ├── cacerts
    │   │   ├── default.policy
    │   │   └── public_suffix_list.dat
    │   ├── server
    │   │   ├── libjsig.so
    │   │   ├── libjvm.so
    │   │   └── Xusage.txt
    │   └── tzdb.dat
    └── release
```







```shell
$ jlink --help
用法: jlink <选项> --module-path <模块路径> --add-modules <模块>[,<模块>...]
可能的选项包括:
      --add-modules <模块>[,<模块>...]    要解析的根模块
      --bind-services                   链接服务提供方模块及其
                                        被依赖对象
  -c, --compress=<0|1|2>                Enable compression of resources:
                                          Level 0: No compression
                                          Level 1: Constant string sharing
                                          Level 2: ZIP
      --disable-plugin <pluginname>     Disable the plugin mentioned
      --endian <little|big>               所生成 jimage
                                          的字节顺序 (默认值: native)
  -h, --help, -?                        输出此帮助消息
      --ignore-signing-information        在映像中链接已签名模块化
                                          JAR 的情况下隐藏致命错误。
                                          已签名模块化 JAR 的签名
                                          相关文件将不会复制到
                                          运行时映像。
      --launcher <名称>=<模块>[/<主类>]
                                        为模块和主类添加给定
                                        名称的启动程序命令
                                        (如果指定)
      --limit-modules <模块>[,<模块>...]  限制可观察模块的领域
      --list-plugins                    List available plugins
  -p, --module-path <路径>                模块路径
      --no-header-files                 Exclude include header files
      --no-man-pages                    Exclude man pages
      --output <路径>                     输出路径的位置
      --post-process-path <imagefile>   Post process an existing image
      --resources-last-sorter <name>    The last plugin allowed to sort
                                        resources
      --save-opts <文件名>                将 jlink 选项保存在指定文件中
  -G, --strip-debug                     Strip debug information
      --suggest-providers [<名称>,...]  建议可从模块路径中实现
                                        给定服务类型的提供方
  -v, --verbose                         启用详细跟踪
      --version                           版本信息
      @<文件名>                           从文件中读取选项
```



## 3.2 任何模块都不是孤岛

### 3.2.1、EasyText实例介绍

### 3.2.2、两个模块



## 3.3、使用平台模块



### 3.3.1、找到正确的平台模块

### 3.3.2、创建GUI模块



## 3.4、封装的限制

