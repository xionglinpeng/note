# Temp-record

JDK 15新特性

Java Development Kit 15是甲骨文公司发布 Java SE（标准版）的最新版本，它在6月11日进入缓降阶段，系列功能现在被冻结。JDK 15的亮点包括文本块、隐藏类、外部内存访问API以及密封类和记录的预览。
Java升级的下一个阶段是另一个缓降阶段，从现在起到8月20日有两个可选版本。预计9月15日正式上市。JDK15紧随3月17日发布的JDK14。甲骨文公司遵循标准Java六个月的发布计划，新版本每年发布两次。

密封类的预览。与接口一起，密封类限制了那些可以扩展或执行的其它类或接口。此特性的目标包括允许类或接口的作者控制由哪些代码负责实现它，并提供比访问修饰符更具声明性的方式来限制超类的使用，还有通过支持对模式的详尽分析来支持模式匹配的未来方向。

记录作为不可变数据的透明载体的类，在jdk14中作为早期预览发布之后，将被包含在jdk15的第二个预览版本中。该计划的目标包括设计一个面向对象构造来表达一个简单的值聚合。以协助程序员专注于不可变数据的建模，而非扩展性行为。自动实现数据驱动的方法，如equals和assessors，并保留Java中长期存在的原则，如名义类型和迁移兼容性。记录可以看作是名义元组。

通过替换java.net.datagram.Socket和java.net.MulticastSocket APIs的实现以更简单和更现代的方式重新实现以前的DatagramSocket API。且易于调试和维护使用项目中当前正在探索的虚拟线程。新计划是JDK增强建议353的后续，该提议重新实现了遗留的Socket API。当前java.net.datagram.Socket和java.net.MulticastSocket的实现可以回溯到jdk1.0，那时IPv6还在开发中。因此，当前的MulticastSocket执行试图以难以维护的方式调节IPv4和IPv6。

隐藏类，即不能被其他类字节码直接使用的类，倾向于借助框架使用，框架会在运行时生成类并通过反射间接使用它们。隐藏类可被定义为访问控制嵌套的成员，并且可以独立于其他类进行卸载。这项提议将提高JVM上所有语言的效率，方法是使用标准API定义不可发现且生命周期有限的隐藏类。



在jdk14和jdk13中预览的文本块旨在简化Java程序编写任务，其可使跨越几行源代码的字符串变得容易表达，同时避免了常见情况下的转义序列。文本块是一个多行字符串文本，它避免了使用过多转义序列，并以可预测的方式自动格式化字符串，并在需要时为开发人员提供对格式的控制。文本块方案的目的是增强Java程序中表示用非Java语言编写字符串的可读性。另一个目标是支持字符串文本的迁移，以确保任何新的构造都可以将同一组字符串表示为字符串文本，解释相同的转义序列，并以与字符串文本相同的方式进行操作。OpenJDK开发人员希望添加转义序列来管理显式的空白和换行控制。Shenandoah low-pause-time垃圾收集器将成为产品的一个特性，并将退出实验阶段。一年前，它已经被集成到jdk12中。
2014年3月在jdk8中首次亮相的Nashorn被移除，由于其被GraalVM等技术淘汰。OpenJDK 15提议要求删除Nashorn APIs和用于调用Nashorn的jjs命令行工具。





手机连接电脑不能被识别：

用手机自带的数据bai线和你的电脑正确连接









1. 首先是启动windows的命令窗口，按键盘上的windows+R，然后在输入框中输入cmd，既可以启动命令窗口

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/cd93a5665159854029fb1351b5a23a42a17ac467.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

2. 

   进入windows命令窗口之后，输入命令，输入netstat -ano然后回车，就可以看到系统当前所有的端口使用情况。

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/3fe32442a07aa01083e2fb8bbfbb19efa35f3e64.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

3. 

   通过命令查找某一特定端口，在命令窗口中输入命令中输入netstat -ano |findstr "端口号"，然后回车就可以看到这个端口被哪个应用占用。

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/a151a233ec3834bb5b3eb5ec8714c27bd3823d64.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

4. 

   查看到对应的进程id之后，就可以通过id查找对应的进程名称，使用命令tasklist |findstr "进程id号"

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/0d55dc7bd2828689e997a00265f97fbd4d7c3764.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

5. 

   通过命令杀掉进程，或者是直接根据进程的名称杀掉所有的进程，，在命令框中输入如下命令taskkill /f /t /im "进程id或者进程名称"

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/87c8bf46b7b1eef9346c5bcfbfb33c4132ba3264.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

6. 

   杀掉对应的进程id或者是进程名称之后，然后再通过查找命令，查找对应的端口，现在就可以看到这个端口没有被其他应用所占用，

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/7efc527c34b33c417c6bc4f2887de137c8762e64.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)