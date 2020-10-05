# JVM



```
[22.971s][info][gc,task     ] GC(36) Using 5 workers of 8 for full compaction
[22.971s][info][gc,start    ] GC(36) Pause Full (System.gc())
[22.971s][info][gc,phases,start] GC(36) Phase 1: Mark live objects
[22.986s][info][gc,phases      ] GC(36) Phase 1: Mark live objects 14.971ms
[22.986s][info][gc,phases,start] GC(36) Phase 2: Prepare for compaction
[22.991s][info][gc,phases      ] GC(36) Phase 2: Prepare for compaction 4.319ms
[22.991s][info][gc,phases,start] GC(36) Phase 3: Adjust pointers
[22.998s][info][gc,phases      ] GC(36) Phase 3: Adjust pointers 7.564ms
[22.998s][info][gc,phases,start] GC(36) Phase 4: Compact heap
[23.004s][info][gc,phases      ] GC(36) Phase 4: Compact heap 5.788ms
[23.011s][info][gc,heap        ] GC(36) Eden regions: 8->0(23)
[23.011s][info][gc,heap        ] GC(36) Survivor regions: 3->0(3)
[23.011s][info][gc,heap        ] GC(36) Old regions: 39->38
[23.011s][info][gc,heap        ] GC(36) Archive regions: 0->0
[23.011s][info][gc,heap        ] GC(36) Humongous regions: 0->0
[23.011s][info][gc,metaspace   ] GC(36) Metaspace: 53280K(54064K)->53280K(54064K) NonClass: 46299K(46768K)->46299K(46768K) Class: 6981K(7296K)->6981K(7296K)
[23.011s][info][gc             ] GC(36) Pause Full (System.gc()) 48M->35M(74M) 39.640ms
[23.011s][info][gc,cpu         ] GC(36) User=0.17s Sys=0.00s Real=0.04s
```







```
[8.402s][info][gc,start    ] GC(33) Pause Young (Concurrent Start) (System.gc())
[8.402s][info][gc,task     ] GC(33) Using 8 workers of 8 for evacuation
[8.405s][info][gc,phases   ] GC(33)   Pre Evacuate Collection Set: 0.1ms
[8.405s][info][gc,phases   ] GC(33)   Merge Heap Roots: 0.1ms
[8.405s][info][gc,phases   ] GC(33)   Evacuate Collection Set: 2.4ms
[8.405s][info][gc,phases   ] GC(33)   Post Evacuate Collection Set: 0.6ms
[8.405s][info][gc,phases   ] GC(33)   Other: 0.2ms
[8.405s][info][gc,heap     ] GC(33) Eden regions: 15->0(20)
[8.405s][info][gc,heap     ] GC(33) Survivor regions: 3->3(3)
[8.405s][info][gc,heap     ] GC(33) Old regions: 39->40
[8.405s][info][gc,heap     ] GC(33) Archive regions: 0->0
[8.405s][info][gc,heap     ] GC(33) Humongous regions: 0->0
[8.405s][info][gc,metaspace] GC(33) Metaspace: 53810K(54684K)->53810K(54684K) NonClass: 46751K(47236K)->46751K(47236K) Class: 7059K(7448K)->7059K(7448K)
[8.405s][info][gc          ] GC(33) Pause Young (Concurrent Start) (System.gc()) 55M->41M(75M) 3.517ms
[8.405s][info][gc,cpu      ] GC(33) User=0.00s Sys=0.00s Real=0.00s
[8.405s][info][gc          ] GC(34) Concurrent Cycle
[8.405s][info][gc,marking  ] GC(34) Concurrent Clear Claimed Marks
[8.405s][info][gc,marking  ] GC(34) Concurrent Clear Claimed Marks 0.057ms
[8.405s][info][gc,marking  ] GC(34) Concurrent Scan Root Regions
[8.408s][info][gc,marking  ] GC(34) Concurrent Scan Root Regions 2.244ms
[8.408s][info][gc,marking  ] GC(34) Concurrent Mark (8.408s)
[8.408s][info][gc,marking  ] GC(34) Concurrent Mark From Roots
[8.408s][info][gc,task     ] GC(34) Using 2 workers of 2 for marking
[8.439s][info][gc,marking  ] GC(34) Concurrent Mark From Roots 31.043ms
[8.439s][info][gc,marking  ] GC(34) Concurrent Preclean
[8.439s][info][gc,marking  ] GC(34) Concurrent Preclean 0.254ms
[8.439s][info][gc,marking  ] GC(34) Concurrent Mark (8.408s, 8.439s) 31.349ms
[8.439s][info][gc,start    ] GC(34) Pause Remark
[8.441s][info][gc          ] GC(34) Pause Remark 41M->41M(75M) 2.263ms
[8.441s][info][gc,cpu      ] GC(34) User=0.00s Sys=0.00s Real=0.00s
[8.441s][info][gc,marking  ] GC(34) Concurrent Rebuild Remembered Sets
[8.480s][info][gc,marking  ] GC(34) Concurrent Rebuild Remembered Sets 38.391ms
[8.480s][info][gc,start    ] GC(34) Pause Cleanup
[8.480s][info][gc          ] GC(34) Pause Cleanup 42M->42M(75M) 0.068ms
[8.480s][info][gc,cpu      ] GC(34) User=0.00s Sys=0.00s Real=0.00s
[8.480s][info][gc,marking  ] GC(34) Concurrent Cleanup for Next Mark
[8.480s][info][gc,marking  ] GC(34) Concurrent Cleanup for Next Mark 0.108ms
[8.480s][info][gc          ] GC(34) Concurrent Cycle 75.133ms
```









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



