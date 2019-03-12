



```shell
C:\Users\dandelion>java -help
用法: java [-options] class [args...]
           (执行类)
   或  java [-options] -jar jarfile [args...]
           (执行 jar 文件)
其中选项包括:
    -d32          使用 32 位数据模型 (如果可用)
    -d64          使用 64 位数据模型 (如果可用)
    -server       选择 "server" VM
                  默认 VM 是 server.

    -cp <目录和 zip/jar 文件的类搜索路径>
    -classpath <目录和 zip/jar 文件的类搜索路径>
                  用 ; 分隔的目录, JAR 档案
                  和 ZIP 档案列表, 用于搜索类文件。
    -D<名称>=<值>
                  设置系统属性
    -verbose:[class|gc|jni]
                  启用详细输出
    -version      输出产品版本并退出
    -version:<值>
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  需要指定的版本才能运行
    -showversion  输出产品版本并继续
    -jre-restrict-search | -no-jre-restrict-search
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  在版本搜索中包括/排除用户专用 JRE
    -? -help      输出此帮助消息
    -X            输出非标准选项的帮助
    -ea[:<packagename>...|:<classname>]
    -enableassertions[:<packagename>...|:<classname>]
                  按指定的粒度启用断言
    -da[:<packagename>...|:<classname>]
    -disableassertions[:<packagename>...|:<classname>]
                  禁用具有指定粒度的断言
    -esa | -enablesystemassertions
                  启用系统断言
    -dsa | -disablesystemassertions
                  禁用系统断言
    -agentlib:<libname>[=<选项>]
                  加载本机代理库 <libname>, 例如 -agentlib:hprof
                  另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help
    -agentpath:<pathname>[=<选项>]
                  按完整路径名加载本机代理库
    -javaagent:<jarpath>[=<选项>]
                  加载 Java 编程语言代理, 请参阅 java.lang.instrument
    -splash:<imagepath>
                  使用指定的图像显示启动屏幕
有关详细信息, 请参阅 http://www.oracle.com/technetwork/java/javase/documentation/index.html。
```





```shell
[root@localhost ~]# jps -help
usage: jps [--help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
    -? -h --help -help: Print this help message and exit.
```



```shell
[root@localhost ~]# jinfo -?
Usage:
    jinfo <option> <pid>
       (to connect to a running process)

where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
    <no option>          to print both VM flags and system properties
    -? | -h | --help | -help to print this help message
```





## jstat

```shell
C:\Users\dandelion>jstat -help
Usage: jstat --help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
  -? -h --help  Prints this help message.
  -help         Prints this help message.
```





```shell
[root@izbp12mpi6ej8nhsdjjk4kz ~]# jmap -?
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> doesnot
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```



| options                | descrition                                                   |
| ---------------------- | ------------------------------------------------------------ |
| `-heap`                | 显示java堆详细信息，例如使用哪种回收器，参数配置，分代状况等。只在Linux和Solaris平台有效。 |
| `-histo`               |                                                              |
| `-clatats`             |                                                              |
| `-finalizerinfo`       |                                                              |
| `-dump:<dump-options>` |                                                              |
| `-F`                   |                                                              |
| `-J<flag>`             |                                                              |
| `-h | -help`           |                                                              |



## Class file structure



```java
CA FE BA BE 00 00 00 37 00 13 0A 00 04 00 0F 09
00 03 00 10 07 00 11 07 00 12 01 00 01 6D 01 00
01 49 01 00 06 3C 69 6E 69 74 3E 01 00 03 28 29
56 010004436F646501000F4C696E654E756D6265725461626C65010003756E6301000328294901000A536F7572636546696C6501000E54657374436C6173732E6A6176610C000700080C0005000601001D636F6D2F66656E697A736F66742F636C617A7A2F54657374436C6173730100106A6176612F6C616E672F4F626A6563740021000300040000000100020005000600000002000100070008000100090000001D00010001000000052AB70001B100000001000A000000060001000000020001000B000C000100090000001F00020001000000072AB400020460AC00000001000A000000060001000000050001000D00000002000E
```









常量池的项目类型

| type                               | tag  | description              |
| ---------------------------------- | ---- | ------------------------ |
| `CONSTANT_Utf8_info`               | 1    | UTF-8编码的字符串        |
| `CONSTANT_Integer_info`            | 3    | 整型字面量               |
| `CONSTANT_Float_info`              | 4    | 浮点型字面量             |
| `CONSTANT_Long_info`               | 5    | 长整型字面量             |
| `CONSTANT_Double_info`             | 6    | 双精度浮点型字面量       |
| `CONSTANT_Class_info`              | 7    | 类或接口的符号引用       |
| `CONSTANT_String_info`             | 8    | 字符串类型字面量         |
| `CONSTANT_Fieldref_info`           | 9    | 字段的符号引用           |
| `CONSTANT_Methodref_info`          | 10   | 类中方法的符号引用       |
| `CONSTANT_InterfaceMethodred_info` | 11   | 接口中方法的引用符号     |
| `CONSTANT_NameAndType_info`        | 12   | 字段或方法的部分引用符号 |
| `CONSTANT_MethodHandle_info`       | 15   | 表示方法句柄             |
| `CONSTANT_MethodType_info`         | 16   | 标识方法类型             |
| `CONSTANT_InvokeDenamic_info`      | 18   | 表示一个动态方法的调用点 |













