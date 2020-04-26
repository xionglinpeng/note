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





