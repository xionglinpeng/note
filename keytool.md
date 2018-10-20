# keytool

https://www.jb51.net/softs/542444.html#downintro2

keytool是一个Java数据证书的管理工具，keytool将密钥（key）和证书（certificates）存在一个称为keystore的文件中。在keystore里，包含两种数据：

- 密钥实体（key entity）—— 密钥（secret key）又或者是私钥和配对公钥（采用非对称加密）
- 可信任的证书实体（trusted certificate entries）—— 只包含公钥

alias（别名）每个keystore都管理这一个独一无二的alias，这个alias统筹不区分大小写。

JDK中keytool常用命令

```shell
[root@localhost]# keytool -help
密钥和证书管理工具

命令:

 -certreq            生成证书请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令

使用 "keytool -command_name -help" 获取 command_name 的用法
```



## -certreq

```shell
[root@localhost]# keytool -certreq -help
keytool -certreq [OPTION]...

生成证书请求

选项:

 -alias <alias>                  要处理的条目的别名
 -sigalg <sigalg>                签名算法名称
 -file <filename>                输出文件名
 -keypass <arg>                  密钥口令
 -keystore <keystore>            密钥库名称
 -dname <dname>                  唯一判别名
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令

使用 "keytool -help" 获取所有可用命令
```

## -genkeypair

```shell
[root@localhost]# keytool -genkeypair -help
keytool -genkeypair [OPTION]...

生成密钥对

选项:

 -alias <alias>                  要处理的条目的别名
 -keyalg <keyalg>                密钥算法名称
 -keysize <keysize>              密钥位大小
 -sigalg <sigalg>                签名算法名称
 -destalias <destalias>          目标别名
 -dname <dname>                  唯一判别名
 -startdate <startdate>          证书有效期开始日期/时间
 -ext <value>                    X.509 扩展
 -validity <valDays>             有效天数
 -keypass <arg>                  密钥口令
 -keystore <keystore>            密钥库名称
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令

使用 "keytool -help" 获取所有可用命令
```



```shell
keytool -genkeypair -alias "test1" -keyalg "RSA" -keystore "test.keystore"  
```



```shell
C:\Users\HP>keytool -genkeypair -alias test -keyalg "RSA" -keystore "test.keystore"
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
  [Unknown]:  localhost
您的组织单位名称是什么?
  [Unknown]:  熊熊科技
您的组织名称是什么?
  [Unknown]:  鹏程万里
您所在的城市或区域名称是什么?
  [Unknown]:  四川成都
您所在的省/市/自治区名称是什么?
  [Unknown]:  四川成都
该单位的双字母国家/地区代码是什么?
  [Unknown]:  zh_CN
CN=localhost, OU=熊熊科技, O=鹏程万里, L=四川成都, ST=四川成都, C=zh_CN是否正确?
  [否]:  y

输入 <test> 的密钥口令
        (如果和密钥库口令相同, 按回车):
再次输入新口令:

Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore test.keystore -destkeystore test.keystore -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。
```





## -genseckey

```shell
[root@localhost]# keytool -genseckey -help
keytool -genseckey [OPTION]...

生成密钥

选项:

 -alias <alias>                  要处理的条目的别名
 -keypass <arg>                  密钥口令
 -keyalg <keyalg>                密钥算法名称
 -keysize <keysize>              密钥位大小
 -keystore <keystore>            密钥库名称
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令

使用 "keytool -help" 获取所有可用命令
```



## -genkey

```shell
[root@localhost]# keytool -genkey -help
keytool -genkeypair [OPTION]...

生成密钥对

选项:

 -alias <alias>                  要处理的条目的别名
 -keyalg <keyalg>                密钥算法名称
 -keysize <keysize>              密钥位大小
 -sigalg <sigalg>                签名算法名称
 -destalias <destalias>          目标别名
 -dname <dname>                  唯一判别名
 -startdate <startdate>          证书有效期开始日期/时间
 -ext <value>                    X.509 扩展
 -validity <valDays>             有效天数
 -keypass <arg>                  密钥口令
 -keystore <keystore>            密钥库名称
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令
```



## -gencert

```shell
[root@localhost]# keytool -gencert -help
keytool -gencert [OPTION]...

根据证书请求生成证书

选项:

 -rfc                            以 RFC 样式输出
 -infile <filename>              输入文件名
 -outfile <filename>             输出文件名
 -alias <alias>                  要处理的条目的别名
 -sigalg <sigalg>                签名算法名称
 -dname <dname>                  唯一判别名
 -startdate <startdate>          证书有效期开始日期/时间
 -ext <value>                    X.509 扩展
 -validity <valDays>             有效天数
 -keypass <arg>                  密钥口令
 -keystore <keystore>            密钥库名称
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令

使用 "keytool -help" 获取所有可用命令
```



## -list

```shell
C:\Users\HP\Desktop>keytool -list -help
keytool -list [OPTION]...

列出密钥库中的条目

选项:

 -rfc                            以 RFC 样式输出
 -alias <alias>                  要处理的条目的别名
 -keystore <keystore>            密钥库名称
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令

使用 "keytool -help" 获取所有可用命令
```

## -printcert

```shell
C:\Users\HP\Desktop>keytool -printcert -help
keytool -printcert [OPTION]...

打印证书内容

选项:

 -rfc                        以 RFC 样式输出
 -file <filename>            输入文件名
 -sslserver <server[:port]>  SSL 服务器主机和端口
 -jarfile <filename>         已签名的 jar 文件
 -v                          详细输出

使用 "keytool -help" 获取所有可用命令
```





keytool -genkey -alias yushan(别名) -keypass yushan(别名密码) -keyalg RSA(算法) -keysize 1024(密钥长度) -validity 365(有效期，天单位) -keystore





spring config 加密

spring cache

keytool

spring boot ssl

redis

doubbo

Dubbo只是实现了服务治理，其他组件需要另外整合以实现对应的功能，比如：
分布式配置：可以使用淘宝的diamond、百度的disconf来实现分布式配置管理。
服务跟踪：可以使用京东开源的Hydra
批量任务：可以使用当当开源的Elastic-Job
而Spring Cloud下面有17个子项目（可能还会新增）分别覆盖了微服务架构下的方方面面，服务治理只是其中的一个方面
2、Dubbo的RPC来实现服务间调用的一些痛点
a、服务提供方与调用方接口依赖方式太强：调用方对提供方的抽象接口存在强依赖关系，需要严格的管理版本依赖，才不会出现服务方与调用方的不一致导致应用无法编译成功等一系列问题；
b、服务对平台敏感，难以简单复用：通常我们在提供对外服务时，都会以REST的方式提供出去，这样可以实现跨平台的特点。
在Dubbo中我们要提供REST接口时，不得不实现一层代理，用来将RPC接口转换成REST接口进行对外发布。所以当当网在dubbox（基于Dubbo的开源扩展）中增加了对REST支持。





https://www.jianshu.com/p/c3879add87ec

https://blog.csdn.net/qq_21909689/article/details/53293116

https://www.cnblogs.com/xdp-gacl/p/3750965.html

https://www.jb51.net/softs/542444.html

https://blog.csdn.net/oMaverick1/article/details/53331380



markdown公式：

https://blog.csdn.net/jyfu2_12/article/details/79207643