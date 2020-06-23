# 动态SQL实现原理

SqlSource接口有四个不同的实现，如下图所示：



1. StaticSqlSource：用于描述ProviderSqlSource、DynamicSqlSource及RawSqlSource解析后得到的静态SQL资源，
2. DynamicSqlSource：用户描述Mapper XML文件中配置的SQL资源信息，这些SQL通常包含动态SQL配置或者`${}`参数占位符，需要在Mapper调用时才能确定具体的SQL语句。
3. RawSqlSource：用于描述Mapper XML文件中配置的SQL资源信息，与DynamicSqlSource不同的是，这些SQL语句在解析XML配置的时候就能确定，即不包含动态SQL相关的配置。
4. ProviderSqlSource：用于描述通过@Select、@SelectProvider等注解配置的SQL资源信息。

无论是Java注解还是XML文件配置的SQL信息，在Mapper调用时都会根据用户传入的参数将Mapper配置转换为StaticSqlSource类。



LanguageDriver接口中一共有三个方法，其源码如下：

```java

```

- ``：
- ``：
- ``：

LanguageDriver接口实现



- `XMLLanguageDriver`：XML语言驱动，为MyBatis提供了通过XML标签（`<if>`、`<where>`等标签）结合OGNL表达式语句实现动态SQL的功能。
- `RawLanguageDriver`：仅支持静态SQL配置，不支持动态SQL功能。







#{}与${}的区别

- 使用`#{}`参数占位符时，占位符内容会被替换成“`?`”，然后通过PreparedStatement对象的`setXXX()`方法为参数占位符设置值；

- 使用`${}`参数占位符时，内容会被直接替换为参数值。

- 使用`#{}`参数占位符能够有效避免SQL注入问题，所以我们可以优先考虑使用`#{}`占位符每当`#{}`参数占位符无法满足需求时，才考虑使用`${}`参数占位符。





