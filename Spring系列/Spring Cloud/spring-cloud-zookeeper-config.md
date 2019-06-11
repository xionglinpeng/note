----

导入配置文件到Zookeeper

配置文件为txt文本，使用之前提到的ZKUI导入到ZooKeeper中。配置文件具体内容如下：

```

/config/demo,dev=spring.datasource.url=jdbc:mysql://192.168.0.176:3306/world?characterEncoding=utf8&useSSL=false
 
/config/demo,dev=spring.datasource.username=root
 
/config/demo,dev=spring.datasource.password=123456
 
/config/demo,dev=spring.datasource.driver-class-name=com.mysql.jdbc.Driver
 
/config/demo,dev=mybatis.mapper-locations=classpath*:/mapper/**Mapper.xml
```

配置命名规则如下：

```
/config/{application-name},{profile}={key}={value}
```

其中{application-name}与{profile}之间用逗号分隔，也可以定义其他分隔符号，在上述bootstrap文件中根据spring.cloud.zookeeper.config.profileSeparator指定