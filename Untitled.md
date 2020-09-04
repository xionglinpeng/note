浏览器在某些请求中，在正式通信前会增加一次HTTP查询请求，称为“预检”请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用那些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则这就报错。



```
Request URL: http://127.0.0.1:8080/data/brands/list
Request Method: OPTIONS
Status Code: 403 
Remote Address: 127.0.0.1:8080
Referrer Policy: no-referrer-when-downgrade
```





```
OPTIONS http://127.0.0.1:8080/index 403 ()
Failed to load http://127.0.0.1:8080/data/brands/list: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8080' is therefore not allowed access. The response had HTTP status code 403.
```







启动docker容器报错`standard_init_linux.go:211: exec user process caused "no such file or directory"`

这个

1、现象

```
standard_init_linux.go:178: exec user process caused "no such file or directory"
```

2、原因

原因是镜像的entrypoint设置的启动脚本格式是dos，在linux系统上用vi修改成unix格式即可

1）用vi打开文件

2）执行 :set ff   然后回车，可以看到fileformat=dos

![img](http://img.wandouip.com/crawler/article/201934/53f832b9fb9d6ebbb74db933f62420d8)

3)修改成unix

:set ff=unix               回车





SpringBoot 解决打出jar包中没有主清单属性

## 原因分析：

在聚合项目中子项目引用了父项目的BOM编译插件，这个BOM代替了**spring-boot-starter-parent**这个parent POM（详见[13.2.2. Using Spring Boot without the parent POM](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-maven-without-a-parent)）

导致spring-boot-maven-plugin的配置项丢失，使得打包后的jar中的MANIFEST.MF文件缺少Main-Class。







![img](https://pics7.baidu.com/feed/8cb1cb13495409236a592348a927eb0cb3de495c.jpeg?token=a5d60e66488219147e7d31e35ea0a105&s=4950E41393C841435C59A8DE000080B2)



<https://www.cnblogs.com/jesse131/p/11529918.html>

<https://www.cnblogs.com/xy51/p/9973326.html>

<https://www.cnblogs.com/bjlhx/p/10477099.html>

<https://blog.csdn.net/blueboz/article/details/105270641>

<https://www.jianshu.com/p/3b91b8958c3e>

<https://www.cnblogs.com/jpfss/p/10945324.html>













## %WINDIR%

系统目录

```shell
C:\Users\chinasoft.lp.xiong> echo %WINDIR%
C:\Windows
```

## %SYSTEMROOT%

系统目录

```shell
C:\Users\chinasoft.lp.xiong> echo %SYSTEMROOT%
C:\Windows
```

## %SYSTEMDRIVE%

系统根目录

```shell
C:\Users\chinasoft.lp.xiong> echo %SYSTEMDRIVE%
C:
```

## %HOMEDRIVE%

当前用户根目录

```shell
C:\Users\chinasoft.lp.xiong> echo %HOMEDRIVE%
C:
```

## %USERPROFILE% 

当前用户目录

```shell
C:\Users\chinasoft.lp.xiong> echo %USERPROFILE%
C:\Users\chinasoft.lp.xiong
```

## %HOMEPATH%

当前用户路径

```shell
C:\Users\chinasoft.lp.xiong> echo %HOMEPATH%
\Users\chinasoft.lp.xiong
```

## %TMP%

当前用户临时文件夹

```shell
C:\Users\chinasoft.lp.xiong> echo %TMP%
C:\Users\CHINAS~1.XIO\AppData\Local\Temp
```

## %TEMP%

当前用户临时文件夹

```shell
C:\Users\chinasoft.lp.xiong> echo %TEMP%
C:\Users\CHINAS~1.XIO\AppData\Local\Temp
```

## %APPDATA%

当前用户数据文件夹

```shell
C:\Users\chinasoft.lp.xiong> echo %APPDATA%
C:\Users\chinasoft.lp.xiong\AppData\Roaming
```

## %PROGRAMFILES%

程序默认安装目录

```shell
C:\Users\chinasoft.lp.xiong> echo %PROGRAMFILES%
C:\Program Files
```

## %COMMONPROGRAMFILES%

文件通用目录

```shell
C:\Users\chinasoft.lp.xiong> echo %COMMONPROGRAMFILES%
C:\Program Files\Common Files
```

## %USERNAME%

当前用户名

```shell
C:\Users\chinasoft.lp.xiong> echo %USERNAME%
chinasoft.lp.xiong
```

## %ALLUSERSPROFILE%

所有用户文件目录

```shell
C:\Users\chinasoft.lp.xiong> echo %ALLUSERSPROFILE%
C:\ProgramData
```

## %OS%

操作系统名

```shell
C:\Users\chinasoft.lp.xiong> echo %OS%
Windows_NT

```

## %COMPUTERNAME%

计算机名

```shell
C:\Users\chinasoft.lp.xiong> echo %COMPUTERNAME%
CD-OFFICE0070
```

## %NUMBER_OF_PROCESSORS%

处理器个数

```shell
C:\Users\chinasoft.lp.xiong> echo %NUMBER_OF_PROCESSORS%
4
```

## %PROCESSOR_ARCHITECTURE%

处理器芯片架构

```shell
C:\Users\chinasoft.lp.xiong> echo %PROCESSOR_ARCHITECTURE%
AMD64
```

## %PROCESSOR_LEVEL%

处理器型号

```shell
C:\Users\chinasoft.lp.xiong> echo %PROCESSOR_LEVEL%
6
```

## %PROCESSOR_REVISION%

处理器修订号

```shell
C:\Users\chinasoft.lp.xiong> echo %PROCESSOR_REVISION%
0f0b
```

## %USERDOMAIN%

包含用户帐号的域

```shell
C:\Users\chinasoft.lp.xiong> echo %USERDOMAIN%
JUSDA
```

## %PATH%

搜索路径

```shell
C:\Users\chinasoft.lp.xiong> echo %PATH%
C:\ProgramData\DockerDesktop\version-bin;C:\Program Files\Docker\Docker\Resources\bin;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;D:\Program Files\Git\cmd;D:\Program Files\gradle\gradle-6.0.1\bin;D:\Program Files\Java\jdk-11.0.2\bin;C:\Users\chinasoft.lp.xiong\AppData\Local\Microsoft\WindowsApps;;D:\Program Files\JetBrains\IntelliJ IDEA 2019.3\bin;
```





