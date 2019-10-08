# Gradle









## 使用Tomcat插件作为Servlet容器

Tomcat插件官方地址：https://plugins.gradle.org/plugin/com.bmuschko.tomcat

GitHup：https://github.com/bmuschko/gradle-tomcat-plugin







```shell
20:52:02: Executing task 'tomcatRun'...

Download https://plugins.gradle.org/m2/com/bmuschko/gradle-tomcat-plugin/2.5/gradle-tomcat-plugin-2.5.jar
> Task :compileJava
> Task :processResources NO-SOURCE
> Task :classes

> Task :tomcatRun
Creation of SecureRandom instance for session ID generation using [SHA1PRNG] took [578] milliseconds.
Started Tomcat Server
The Server is running at http://localhost:8080/learn-servlets
```





gradle配置tomcat插件



```shell
> Task :compileJava NO-SOURCE
> Task :processResources NO-SOURCE
> Task :classes UP-TO-DATE
Download https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-jasper/7.0.59/tomcat-embed-jasper-7.0.59.pom
Download https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-core/7.0.59/tomcat-embed-core-7.0.59.pom
Download https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-logging-juli/7.0.59/tomcat-embed-logging-juli-7.0.59.pom
Download https://repo.maven.apache.org/maven2/org/eclipse/jdt/core/compiler/ecj/4.4/ecj-4.4.pom
Download https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-el/7.0.59/tomcat-embed-el-7.0.59.pom
Download https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-logging-juli/7.0.59/tomcat-embed-logging-juli-7.0.59.jar
Download https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-jasper/7.0.59/tomcat-embed-jasper-7.0.59.jar
Download https://repo.maven.apache.org/maven2/org/eclipse/jdt/core/compiler/ecj/4.4/ecj-4.4.jar
Download https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-el/7.0.59/tomcat-embed-el-7.0.59.jar
Download https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-core/7.0.59/tomcat-embed-core-7.0.59.jar
> Task :tomcatRun
Started Tomcat Server
The Server is running at http://localhost:8080/learn-servlet
```

