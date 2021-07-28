



# [Jackson 格式化日期问题](https://www.cnblogs.com/njl041x/p/6235966.html)



Jackson 默认是转成timestamps形式的，如何使用自己需要的类型，

解决办法： 1、在实体字段上使用@JsonFormat注解格式化日期

```javascript
@JsonFormat(locale="zh", timezone="GMT+8", pattern="yyyy-MM-dd HH:mm:ss")
```

2、通过下面方式可以取消timestamps形式

```java
objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
```

自定义输出格式

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
objectMapper.setDateFormat(sdf)
```