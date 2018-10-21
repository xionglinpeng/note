# keytool



证书类型

| 格式   | 扩展名         | 描述                                                    | 特点                                                         |
| ------ | -------------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| DER    | .cer/.crt/.rsa | 【ASN.1 DER】用于存放证书                               | 不含私钥，二进制                                             |
| PKCS7  | .p7b/.p7r      | 【PKCS #7】加密信息语法标准                             | 1、p7b以树状展示证书链，不含私钥<br/>2、p7r为CA对证书请求签名的回复，只能用于导入 |
| CMS    | .p7c/.p7m/.p7s | 【Cryptographic Message Syntax】                        | 1、p7c只保存证书<br />2、p7m：signature with enveloped data <br />3、p7s：时间戳签名文件 |
| PEM    | .pem           | 【Printable Encoded Message】                           | 1、该编码格式在RFC1421中定义，其实PEM是【Privacy-Enhanced Mail】的简写，但它也同样广泛运用于秘钥管理。<br />2、ASCII文件<br />3、一般基于base 64编码 |
| PKCS10 | .p10/.csr      | 【PKCS #10】公钥加密标准【Certificate Signing Request】 | 1、证书签名请求文件<br />2、ASCII文件<br />3、CA签名后以p7文件回复 |
| SPC    | .pvk/.spc      | 【Software Publishing Certificate】                     | 微软公司特有的双证书文件格式，经常用于代码签名，其中<br />1、pvk用于保存私钥<br />2、spc用于保存公钥 |
|        |                |                                                         |                                                              |



## 证书

证书是来自某个实体的经数字签名的声明，它声明另一个实体的公钥具有某一特定的值。

## 公钥

## 身份

## 签名

所谓签名，就是用实体的（签名人，在证书中也称为签发人）私钥对某些数据进行计算。

### 私钥

是一些数字，每个数字都对应仅被以该数字作为私钥的特定实体所知。在所有公钥密码系统中，私钥和公钥均成对出现，在DSA等具体的公钥密码系统中，一个私钥只对应一个公钥，私钥用于计算签名。

### 实体

实体是您在某种程度上对其加以信任的个人、组织、程序、计算机、企业、银行等。

通常，公钥密码系统需要访问用户的公钥。在大型互联网环境中，并不能确保通信实体之间已经预先建立起联系，也无法确保受信任的存储库与所用的公钥都存在。于是人们发明了证书作为公钥分配问题的解决办法。现在，认证机构（CA）可充当可信任的第三方。



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

keytool -genkeypair -alias "star-travel-guest-config-server" -keystore "D:\program\Workspace\study\star-travel-guest\star-travel-guest-config-server\src\main\resources\config-server.keystore" -storepass "129f7cab-2bd2-48c6-ad02-d5931f3a8862" -keypass "123456" -keyalg "RSA"
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