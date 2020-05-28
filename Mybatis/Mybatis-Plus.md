# Mybatis-Plus



```java
@TableId(value = "id", type = IdType.AUTO)
private Long id;
@TableId(value = "uuid", type = IdType.UUID)
private String uuid;
```





```java
package com.baomidou.mybatisplus.annotation;

import lombok.Getter;

/**
 * 生成ID类型枚举类
 * @author hubin
 * @since 2015-11-10
 */
@Getter
public enum IdType {
    /**
     * 数据库ID自增
     */
    AUTO(0),
    /**
     * 该类型为未设置主键类型
     */
    NONE(1),
    /**
     * 用户输入ID
     * 该类型可以通过自己注册自动填充插件进行填充
     */
    INPUT(2),

    /* 以下3种类型、只有当插入对象ID 为空，才自动填充。 */
    /**
     * 全局唯一ID (idWorker)
     */
    ID_WORKER(3),
    /**
     * 全局唯一ID (UUID)
     */
    UUID(4),
    /**
     * 字符串全局唯一ID (idWorker 的字符串表示)
     */
    ID_WORKER_STR(5);

    private int key;

    IdType(int key) {
        this.key = key;
    }
}

```





```java
Caused by: org.apache.ibatis.reflection.ReflectionException: Could not set property 'uuid' of 'class com.threes.city.entity.City' with value '1051711067145351169' Cause: java.lang.IllegalArgumentException: argument type mismatch
```





```
@TableField("`delete`")
```





```java
package com.baomidou.mybatisplus.annotation;

/**
 * 字段填充策略枚举类
 * @author hubin
 * @since 2017-06-27
 */
public enum FieldFill {
    /**
     * 默认不处理
     */
    DEFAULT,
    /**
     * 插入填充字段
     */
    INSERT,
    /**
     * 更新填充字段
     */
    UPDATE,
    /**
     * 插入和更新填充字段
     */
    INSERT_UPDATE
}

```





自定义模板

```java
16:10:18.767 [main] DEBUG freemarker.cache - Couldn't find template in cache for "D:\\workspace\\threes-server\\threes\\common\\src\\main\\resources\\entity.java.ftl.ftl"("zh_CN", UTF-8, parsed); will try to load it.
16:10:18.767 [main] DEBUG freemarker.cache - TemplateLoader.findTemplateSource("D:\\workspace\\threes-server\\threes\\common\\src\\main\\resources\\entity.java.ftl_zh_CN.ftl"): Not found
16:10:18.768 [main] DEBUG freemarker.cache - TemplateLoader.findTemplateSource("D:\\workspace\\threes-server\\threes\\common\\src\\main\\resources\\entity.java.ftl_zh.ftl"): Not found
16:10:18.769 [main] DEBUG freemarker.cache - TemplateLoader.findTemplateSource("D:\\workspace\\threes-server\\threes\\common\\src\\main\\resources\\entity.java.ftl.ftl"): Not found
```





自定义模板

自定义的模板放置在classpath下即可，以绝对路径设置，例如`/templates/entity.java`。

```java
        /*-------------------- 模板配置 --------------------*/
        //模板配置，可自定义代码生成的模板，实现个性化操作
        TemplateConfig templateConfig = new TemplateConfig();
//        templateConfig.setController("");
        templateConfig.setEntity("/templates/entity.java");
//        templateConfig.setEntityKt("");
//        templateConfig.setMapper("");
//        templateConfig.setService("");
//        templateConfig.setServiceImpl("");
//        templateConfig.setXml("");
        mpg.setTemplate(templateConfig);
```



## 逻辑删除

```yml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```



```java
@Bean
public ISqlInjector sqlInjector() {
    return new LogicSqlInjector();
}
```



```java
@TableLogic
private Integer deleted;
```



```sql
SELECT `uuid`,`name` FROM user WHERE `uuid`='5d0301a9ac9d44999e3493ee704792d6' AND `deleted`=1

UPDATE user SET `deleted`=0 WHERE `uuid`='5d0301a9ac9d44999e3493ee704792d6' AND `deleted`=1
```

