# 不同JDK发行版

## OpenJDK与OracleJDK区别

在Java 11之后，OracleJDK和OpenJDK的功能基本一致，之前OracleJDK中私有的组件大多数以及被捐赠给开源组织了，现在它们之间只有少量的区别。

要比较那个更稳定，OpenJDK实际上不适合拿来和OracleJDK进行比较，OpenJDK不提供LTS服务，而OracleJDK每三年都会推出一个LTS版进行长期支持。和OracleJDK对比的应该是AdoptOpenJDK、Liberica JDK、Zulu OpenJDK以及Red Hat OpenJDK，它们都是基于OpenJDK的发行版，由不同的商业公司提供商业支持，包括和OracleJDK周期相同的LTS版。与这些OpenJDK的发行版相比，OracleJDK并没有本质的差异，稳定性也是差不多的。

相比之下，更推荐选择一个OpenJDK的发行版使用，而不是选择OracleJDK：

- OracleJDK和这些OpenJDK发行版的功能基本一致，背后同样有公司提供商业支持，稳定性也难分优劣，Red Hat OpenJDK、Zulu OpenJDK和Liberica JDK都通过了TCK。
- Oracle修改了协议，除了开发、测试以及演示用途，其他场合都是需要收费的，而AdoptOpenJDK、Liberica JDK和Zulu OpenJDK都提供适用于各种用途的免费版本，并提供LTS服务，只有在需要商业支持的时候需要付费。
- OracleJDK对一般用户友善程度也不是最高的，新版不在捆绑JavaFX，同时不提供32位构建。

现在推荐开发者和一般用户使用Liberica JDK。

**差异**

- **授权协议不同**：OpenJDK采用GPL V2协议放出，而SUN JDK则采用JRL放出。两种协议虽然都是开放源代码的，但是在使用上的不同在于GPL V2允许在商业上使用，而JRL只允许个人研究使用。
- **OpenJDK源代码不完整**：在采用GPL协议的OpenJDK中，Oracle JDK的一部分源代码因为产权的问题无法开放给OpenJDK使用，其中最主要的部分就是JMX中的可选元件SNMP部分的代码。这些代码被制作成plugin，以供OpenJDK编译时使用（可以选择不使用）。icedtea为这些不完整的部分开发了相同功能的源代码（OpenJDK 6）。
- **部分源代码用开源代码替换**：由于产权问题，很多产权不是Oracle的源代码被替换成一些功能相同的开源代码，比如说字体栅格化引擎，使用Free Type代替。
- **OpenJDK只包含最精简的JDK**：OpenJDK不包含其他软件包，比如Rhino Java DB JAXP等。

**相关资料**

- [Oracle JDK Releases for Java 11 and Later](https://blogs.oracle.com/java-platform-group/oracle-jdk-releases-for-java-11-and-later)

## OpenJDK

Official：http://jdk.java.net/

## AdoptOpenJDK

AdoptOpenJDK是OpendJDK的社区维护版，主要维护8,11两个LTS版本以及最新版本。

如果要使用Shenandoah垃圾收集器就需要使用AdoptOpenJDK，而在OracleJDK和OpenJDK上是不包含Shenandoah垃圾收集器的。

Official：https://adoptopenjdk.net/

## Liberica JDK

Official：https://bell-sw.com/

Liberica JDK提供了捆绑JavaFX的Full版，支持macOS x86_64、Windows  x86、Windows x86_64、Linux x86、Linux x86_64、Alpine Linux x86_64、Linux ARMv8、Linux ARMv7 HardFloat、Linux PPC64 LE等平台，对于需要32位环境或者需要在小众平台上工作的用户来说很友好。同时它的exe安装包能够方便的配置环境变量，还提供yum和apt仓库，提供Docker镜像，目前来说应该是对一般用户最友好的OpenJDK发行版。

## Zulu OpenJDK



Official：https://www.azul.com/

## Red Hat OpenJDK

Official：https://developers.redhat.com/products/openjdk/download

