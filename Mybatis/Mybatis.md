# Mybatis

## mybatis高级查询

### 高级结果映射

#### 6.1.1、一对一映射

通过一次查询将结果映射到不同对象的方式，称之为关联的前台结果映射。

关联的嵌套结果映射需要关联多个表将所有需要的值一次性查询出来。这种方式的好处是减少数据库查询次数，减轻数据库的压力，缺点是要写很复杂的SQL，并且当嵌套结果更复杂时，不容易一次写正确，由于要在应用服务器上将结果映射到不同的类上，因此也会增加应用服务器的压力。当一定会使用到嵌套结果，并且整个复杂的SQL执行速度很快时，建议使用关联嵌套结果查询。

6.1.1.2、使用`<resultMap>`配置一对一映射

6.1.1.3、使用`<resultMap>`的`<association>`标签配置一对一映射

`<association>`标签包含以下属性：

- `property`： 对应实体类中的属性名，必填项。
- `javaType`： 属性对应的java类型。
- `resultMap`： 可以直接使用现有的`resultMap`，而不需要在这里配置。
- `columnPrefix`： 查询列的前缀，配置前缀后，在子标签配置`<result>`的`column`时可以省略前缀。

6.1.1.4、`<association>`标签的嵌套查询

`<association>`标签的嵌套查询常用的属性如下：

- `select`：另一个映射查询的id，MyBatis会额外执行这个查询获取嵌套对应的结果。
- `column`：列名（或别名），将主查询中列的结果作为嵌套查询的参数，配置方式如`column={prop=col1,prop2=col2}`，`prop1`和`prop2`将作为嵌套查询的参数。
- `fetchType`：数据加载方式，可选值为`lazy`和`eager`，分别为延迟加载和积极加载，这个配置会覆盖全局的`lazyLoadingEnabled`配置。



特别提醒！

> ​	许多对延迟加载原理不太熟悉的朋友经常会遇到一些莫名其妙的问题：有些时候延迟加载可以得到数据，有些时候延迟加载就会报错，为什么会出现这种情况呢？
>
> ​	MyBatis延迟加载是通过动态代理实现的，当调用配置为延迟加载的属性方法时，动态代理的操作会被触发，这些额外的操作就是通过MyBatis的`SqlSession`去执行嵌套SQL的。由于在和某些框架集成时，`SqlSeesion`的生命周期交给了框架来管理，因此当对象超出`SqlSession`的生命周期调用时，会由于链接关闭等问题而抛出异常。在和Spring集成时，要确保只能在Service层调用延迟加载的属性。当结果从Service层返回至Controller层时，如果获取延迟加载的属性值，会因为`SqlSession`已经关闭而抛出异常。



#### 6.1.2、一对多映射

##### 6.1.2.1、collection集合的嵌套结果映射

和`association`类似，集合的嵌套结果映射就是指通过一次SQL查询将所有的结果查询出来，然后通过配置的结果映射，将数据映射到不同的对象中去。在一对多的关系中，主表的一条数据会对应关联表的多条数据库，因此一般查询时会查询出多个结果，按照一对多的数据库结构存储数据的时候，最终的结果数会小于等于查询的总记录数。



```xml
<resultMap id="BaseResultMap" type="com.threes.city.entity.City">
    <id column="id" property="id" />
    <id column="uuid" property="uuid" />
    <result column="name" property="name" />
    <result column="serial_number" property="serialNumber" />
    <result column="remark" property="remark" />
    <result column="deleted" property="deleted" />
    <result column="create_time" property="createTime" />
    <result column="update_time" property="updateTime" />
</resultMap>

<resultMap id="allCitysResultMap" extends="BaseResultMap" type="com.threes.city.entity.City">
    <collection property="districts" columnPrefix="district_"
 resultMap="com.threes.city.mapper.DistrictMapper.allDistrictResultMap"/>
</resultMap>
```



```xml
<resultMap id="BaseResultMap" type="com.threes.entity.city.District">
    <id column="id" property="id" />
    <id column="uuid" property="uuid" />
    <result column="create_time" property="createTime" />
    <result column="update_time" property="updateTime" />
    <result column="city_uuid" property="cityUuid" />
    <result column="name" property="name" />
    <result column="serial_number" property="serialNumber" />
    <result column="remark" property="remark" />
    <result column="deleted" property="deleted" />
</resultMap>

<resultMap id="allDistrictResultMap" extends="BaseResultMap" type="com.threes.entity.city.District">
    <collection property="children" columnPrefix="circle_"
                resultMap="com.threes.city.mapper.DistrictCircleMapper.BaseResultMap"/>
</resultMap>
```





```mysql
<select id="selectAllCities" resultMap="allCitysResultMap">
    SELECT
        c.`uuid`,
        c.`name`,
        c.`serial_number`,
        c.`remark`,
        cd.`uuid` AS district_uuid,
        cd.`city_uuid` AS district_city_uuid,
        cd.`name` AS district_name,
        cd.`serial_number` AS district_serial_number,
        cd.`remark` AS district_remark,
        cdc.`uuid` AS district_circle_uuid,
        cdc.`city_district_uuid` AS district_circle_city_district_uuid,
        cdc.`name` AS district_circle_name,
        cdc.`serial_number` AS district_circle_serial_number,
        cdc.`remark` AS district_circle_remark
    FROM
    	`city` c
        LEFT JOIN `city_district` cd
        ON c.`uuid` = cd.`city_uuid`
        LEFT JOIN `city_district_circle` cdc
        ON cd.`uuid` = cdc.`city_district_uuid`
    WHERE c.`deleted` = 1 AND cd.`deleted` = 1 AND cdc.`deleted` = 1
</select>
```





- `property`: 
- `columnPrefix`: 
- `resultMap`: 

理解Mybatis处理的规则对使用一对多配置是非常重要的，如果只是一知半解，很容易就会遇到各种莫名其妙的问题，所以针对Mybatis处理中的要点，下面进行一个详细的阐述：

Mybatis在处理结果的时候，会判断结果是否相同，如果是相同的结果，则只会保留第一个结果，所以这个问题的关键点就是Mybatis如何判断结果是否相同。Mybatis判断结果是否相同时，最简单的情况就是在映射配置中只是有一个`id`标签。

例如：

```xml
<id column="id" property="id" />
```



我们对`id`的理解一般是，它配置的字段为表的主键（联合主键时可以配置多个`id`标签）因为**Mybatis的`resultMap`只用于配置结果如何映射，并不知道这个表具体如何。`id`的唯一作用就是在嵌套的映射配置时判断数据是否相同**，当配置`id`标签时，Mybatis只需要逐条比较所有数据中`id`标签配置的字段值是否相同即可。在配置嵌套结果查询时，配置`id`标签可以提高处理效率。

如果没有配置`id`时，Mybatis就会把`resultMap`中配置的所有字段进行比较，如果所有字段的值都相同就合并，只要有一个字段值不同，就不合并。但是由于Mybatis要对所有字段进行比较，因此当字段数为M时，如果查询的结果有N条，就需要进行M$$\times$$N次比较，相比配置`id`时的N次比较，效率相差更多，所以要尽可能配置`id`标签。

**注意：**

> 在嵌套结果配置id属性时，如果查询语句中没有查询id属性配置的列，就会导致id对应的值为null。这种情况下，所有值的id都相同，因此会使嵌套的集合中只有一条数据。所以在配置id列时，查询语句中必须包含改列。



注意，如果有多个嵌套嵌套映射，比如上面的例子，其别名需要前缀需要叠加，因为每一层嵌套映射都是在上一层的基础上进行比较映射的，而不是在根嵌套上进行映射的。

比如上面的例子，第一层前缀是`district_`，第二层就是`district_circle_`，即是说在第二层嵌套映射的时候，已经在第一次将前缀为``district_`的别名前缀都去掉了。如果有第三层，第四层等等，依次类推。

它不仅可以被嵌套的配置引用，其本身也可以使用。一个复杂的映射就是由这样一个基本的映射配置组成的。通常情况下，如果要配置一个相当复杂的映射，一定要从基础映射开始配置，每增加一些配置就进行对应的测试，在循序渐进的过程中更容易发现和解决问题。

虽然`collection`和`association`标签是分开介绍的，但是这两者可以组合使用或者相互嵌套使用，也可以使用符合自己需要的任何数据结构，不需要局限于数据库表之间的关联关系。

##### 6.1.2.2、collection集合的嵌套查询

`collection`集合的嵌套查询的配置方式跟`association`是一样的，例子如下：

```xml
<resultMap id="CurriculaumResultMap" extends="BaseResultMap" type="com.threes.entity.curriculaum.Curriculaum">
    <association property="detail" fetchType="lazy" column="{curriculaumUuid=uuid}"
select="com.threes.curriculaum.mapper.CurriculaumDetailMapper.selectCurriculaumDetailById"/>
    <collection property="images" fetchType="lazy" column="{curriculaumUuid=uuid}"
  select="com.threes.curriculaum.mapper.CurriculaumImagesMapper.selectCurriculaumImagesById"/>
</resultMap>
```



```xml
<select id="selectCurriculaums" resultMap="CurriculaumResultMap" parameterType="com.threes.entity.vo.CurriculaumSelectDto">
    SELECT
        create_time,
        update_time,
        uuid, organization_uuid, serial_number, name, logo_image, original_price,
        now_price, suit_throng_start_age, suit_throng_end_age,
        suit_basics, attend_class_number_start,attend_class_number_end,
        number_browser, number_intention, putaway
    FROM
      `curriculaum`
    <where>
        `deleted` = 1
        <if test="cur.organizationUuid!=null">
            AND organization_uuid = #{cur.organizationUuid}
        </if>
    </where>
    ORDER BY `create_time` DESC,
      `putaway` DESC
</select>
```



```xml
<select id="selectCurriculaumDetailById" resultMap="BaseResultMap">
    SELECT
    detail_url,
    detail_html
    FROM
    curriculaum_detail
    WHERE `curriculaum_uuid` = #{curriculaumUuid}
</select>
```



```xml
<select id="selectCurriculaumImagesById" resultMap="BaseResultMap">
    SELECT
    image_url
    FROM
    `curriculaum_images`
    WHERE curriculaum_uuid = #{curriculaumUuid}
</select>
```

这里需要注意：

- `collection`的属性`column`配置为`{curriculaumUuid=uuid}`，将当前主查询的`uuid`赋值给`curriculaumUuid`，使用`curriculaumUuid`作为嵌套SQL的参数进行查询。因为所有的嵌套查询都配置为延迟加载（`fetchType="lazy"`），因此不存在N+1的问题。
- 之所以可以根据需要查询数据，除了和fetchType有关，还和全局的`aggressive-lazy-loading`属性有关，这个属性在介绍`association`时被配置成了false，所以才会起到按需加载的作用。



#### 鉴别器映射

有时一个单独的数据库查询会返回很多不同的数据类型（希望有些关联）的结果集。``鉴别器标签就是用来处理这种情况的。鉴别器非常容易理解，因为它很想Java语言中的switch语句。简单来说就是根据主SQL查询的结果作为判断条件，而执行不同的嵌套SQL。

`<discriminator>`标签常用的两个属性如下：

- `column`：该属性用于设置要进行鉴别比较值的列。
- `javaType`：该属性用于指定列的类型，保证使用相同的Java类型来比较值。

`<discriminator>`标签可以有1个或多个`<case>`标签，`<case>`标签包含以下三个属性：

- `value`：该值为`<discriminator>`指定`column`用来匹配的值。
- `resultMap`：当`column`的值和`value` 的值匹配时，可以配置使用`resultMap`指定的映射，`resultMap`优先级高于`resultType`。
- `resultType`：当`column`的值和`value`的值匹配时，用于配置使用`resultType`指定的映射。

`<case>`标签下面可以包含的标签和`resultMap`一样，用法也一样。

### 6.3、使用枚举映射

假设在User表中存在一个名叫enabled的字段，这个字段只有两个可选值，0为禁用，1位启用。但是在映射的实体类中，我们使用的是`private Integer enable`，这种情况下必须手动校验enabled的值是否符合要求，因为用户卡能传来2、3、...等。在只有两个值的情况下，处理起来还比较容易，但是当出现更多的可选值时，对值进行校验就会变得复杂。因此我们可以使用枚举映射来解决这个问题。

myabtis默认使用了`org.apache.ibatis.type.EnumTypeHandler`枚举处理器，这个枚举处理器是以字面值进行映射。

还提供了`org.apache.ibatis.type.EnumOrdinalTypeHandler`枚举处理器，这个枚举处理器是以枚举索引数字进行映射。

#### 6.3.1、使用Mybatis提供的枚举处理器

如果我们只是需要枚举字面值映射，那么不需要做任何配置，如果需要其他方式的映射，可以如下配置：

- mybatis-config.xml配置，指定指定的枚举对应的类型处理器

  ```xml
  <typeHandlers>
  	<typeHandler javaType="*.**.Enabled" handler="org.apache.ibatis.type.EnumOrdinalTypeHandler"/>
  </typeHandlers>
  ```

- 或者可以在xml文件中指定对应的枚举字段类型处理器：

  ```xml
  <resultMap id="BaseResultMap" type="...">
      <id column="id" property="id" />
      ......
      <result column="deleted" property="deleted" 		typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler"/>
  </resultMap>
  ```

- 也可以通过SpringBoot配置指定默认的枚举处理器：

  ```yaml
  mybatis:
    configuration:
      default-enum-type-handler: org.apache.ibatis.type.EnumOrdinalTypeHandler
  ```

#### 6.3.2、自定义类型处理器

有些时候Mybatis提供的类型处理器并不能满足我们的需求，这个时候就只能自定义类型处理器了。Mybatis提供了`org.apache.ibatis.type.TypeHandler`接口，通过实现此接口就可以编写自己的类型处理器了。

TypeHandler接口源码如下：

```java
package org.apache.ibatis.type;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
/**
 * @author Clinton Begin
 */
public interface TypeHandler<T> {
  void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;
  T getResult(ResultSet rs, String columnName) throws SQLException;
  T getResult(ResultSet rs, int columnIndex) throws SQLException;
  T getResult(CallableStatement cs, int columnIndex) throws SQLException;
}
```

它有四个抽象方法：

- `setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType)`：设置参数的回调的。
- `getResult(ResultSet rs, String columnName)`：通过字段名从结果集中获取值。
- `getResult(ResultSet rs, int columnIndex)`：通过字段索引从结果集中获取值。
- `getResult(CallableStatement cs, int columnIndex)`：通过字段索引从`CallableStatement`获取值。

在TypeHandler接口实现类中，除了默认的无参构造方法，还有一个隐含的带有一个Class参数的构造方法。

```java
public xxxTypeHandler(Class<?> type) {
    this();
}
```

当针对特定的接口处理类型时，使用这个构造方法可以写出通用的类型处理器，就像Mybatis提供的两个枚举类型处理器一样。

有了自己的类型处理器之后还需要配置，配置方式就跟前面描述的枚举处理器的配置方式一样（所有的类型处理器配置方式都是一样的）。

下面是一个例子：

数据库字段为一个VARCHAR类型，存储的是一个枚举类型字面值以逗号分隔的字符串，实体类型为枚举的List集合类型。现在要将这个字符串映射为对应枚举的List集合。

```java
package com.threes.curriculaum.type;

import com.threes.enums.SuitBasics;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.TypeHandler;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class SuitBasicsTypeHandler implements TypeHandler<List<SuitBasics>> {

    @Override
    public void setParameter(PreparedStatement ps, int i, List<SuitBasics> parameter, JdbcType jdbcType) throws SQLException {
        String suitBasics = parameter.stream().map(SuitBasics::toString).collect(Collectors.joining(","));
        ps.setString(i,suitBasics);
    }

    @Override
    public List<SuitBasics> getResult(ResultSet rs, String columnName) throws SQLException {
        String suitBasics = rs.getString(columnName);
        return Arrays.stream(suitBasics.split(",")).map(SuitBasics::valueOf).collect(Collectors.toList());
    }

    @Override
    public List<SuitBasics> getResult(ResultSet rs, int columnIndex) throws SQLException {
        String suitBasics = rs.getString(columnIndex);
        return Arrays.stream(suitBasics.split(",")).map(SuitBasics::valueOf).collect(Collectors.toList());
    }

    @Override
    public List<SuitBasics> getResult(CallableStatement cs, int columnIndex) throws SQLException {
        String suitBasics = cs.getString(columnIndex);
        return Arrays.stream(suitBasics.split(",")).map(SuitBasics::valueOf).collect(Collectors.toList());
    }
}

```

注意：这个类型映射器是用于从数据库到实体的类型映射转换，而不是从实体到数据库的映射转换。所以在插入数据库的时候，为实体类的List枚举类型字段赋的值不能插入数据库，需要将List枚举转换为字符串。那么Mybatis是如何转换的呢？Mybatis在从实体类型映射到`insert`语句时是通过反射调用了对应实体字段的`get`方法，所以只需要重写`get`方法，在`get`方法中自己实现转换即可。

mybatis-plus`@TableField`注解提供了`el`属性指定mybatis映射`insert`时调用的方法（就不会调用对应的`get`方法了，好处是仍然可以保留`get`方法取值时的方便），如下`suitBasicsMapping`：

```java
@TableField(value = "`suit_basics`",el = "suitBasicsMapping, jdbcType=VARCHAR")
private List<SuitBasics> suitBasics;

public String getSuitBasicsMapping() {
    return suitBasics.stream().map(SuitBasics::toString).collect(Collectors.joining(","));
}
```

## MyBatis缓存配置

### 一级缓存

​	MyBatis的一级缓存存在于`SqlSession`的生命周期中，在同一个SqlSession中查询时，MyBatis会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个Map对象中。如果同一个`SqlSession`中执行的方法和参数完全一致，那么通过算法会生成相同的键值。当Map缓存对象中已经存在该键值时，则会返回缓存中的对象。

如果不想让指定的select方法使用一级缓存，可以添加`flushCache`属性。

```xml
<select id=".." flushCache="true" resultMap="..">
    ...
</select>
```

这个属性配置为`true`后，会在查询数据前清空当前的一级缓存，因此该方法每次都会重新从数据库中查询数据。但是由于这个方法清空了一级缓存，影响当前`SqlSession`中所有缓存的查询，因此在需要反复查询获取只读数据的情况下，会增加数据库的查询次数，所以要避免这么使用。

`INSERT`、`UPDATE`、`DELETE`操作都会情况一级缓存。

### 二级缓存

#### 配置二级缓存

在MyBatis的全局配置`<settings>`中有一个参数`cacheEnabled`，这个参数是二级缓存的全局开关，默认值是`true`，初始状态为启用状态。如果把这个参数设置为`false`，即使有后面的二级缓存配置，也不会生效。由于这个参数值默认为`true`，所以不必配置，如果想要配置，可以在`mybatis-config.xml`中添加如下代码：

```xml
<settings>
	...
    <setting name="cacheEnabled" value="true"/>
    ...
</settings>
```

MyBatis的二级缓存是和命名空间绑定的，即二级缓存需要配置在`Mapper.xml`映射文件中，或者配置在`Mapper.java`接口中。在映射文件中，命名空间就是XML根节点`<mapper>`的`namespace`属性。在Mapper接口中，命名空间就是接口的全限定名称。

##### `Mapper.xml`中配置二级缓存



默认的二级缓存会有如下效果：

- 映射语句文件中的所有`SELECT`语句将会被缓存。
- 映射语句文件中的所有`INSERT`、`UPDATE`、`DELETE`语句会刷新缓存。
- 缓存会使用Least Recently Used（LRU，最近最少使用的）算法来回收。
- 根据时间表（如no Flush Interval，没有刷新间隔），缓存不会以任何时间顺序来刷新。
- 缓存会存储集合或对象（无论查询方法返回什么类型的值）的个引用。
- 缓存会被视为read/write（可读/可写）的，意味着对象检索不是共享的，而且可以安全地被调用者修改。而不干扰其他调用者或者线程所做的潜在修改。

所有的这些属性都可以通过缓存元素的属性来修改，如下：

```xml
<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>
```



`<cache>`配置属性：

- `eviction`：回收策略。
  - `LRU`（最近最少使用的）：移除最长时间不被使用的对象，这是默认值。
  - `FIFO`（先进先出）：按对象进入缓存的顺序来移除它们。
  - `SOFT`（软引用）：移除基于垃圾回收器状态和软引用规则的对象。
  - `WEAK`（若引用）：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
- `flushInterval`：（刷新间隔）。可以被设置为任意的正整数，而且它们代表一个合理的毫秒形式的时间段。默认情况不设置，即没有刷新间隔，缓存仅仅在调用语句时刷新。
- `size`：（引用数目）。可以被设置为任意正整数，要记住缓存的对象数目和运行环境的可用内存资源数目相关。默认值是1024。
- `readOnly`：（只读）。属性可以被设置为`true`或`false`。只读的缓存会给所有调用者返回缓存对象的相同实例，因此这些对象不能被修改，这提供了很重要的性能优势。可读写的缓存会通过序列化返回缓存对象的拷贝，这种方式会慢一下，但是安全，因此默认是false。

##### `Mapper`接口中配置二级缓存

在Mapper接口配置二级缓存，只需要在Mapper接口上添加`@CacheNamespace`注解即可。

```java
import org.apache.ibatis.annotations.CacheNamespace;
import org.apache.ibatis.cache.decorators.FifoCache;
@CacheNamespace
public interface Mapper {
}
```

它同样具有与XML配置一样的属性，示例如下：

```java
@CacheNamespace(eviction = FifoCache.class,flushInterval = 60000,size = 512,readWrite = true)
```

注意：

> 这里的`readWrite`属性与XML中的`readOnly`属性一样，用于配置缓存是否为只读类型，在这里`true`为读写，`false`为只读。默认为`true`。

注意：如果XML和Mapper接口同时开启了二级缓存会怎样？

> 如果同时开启了二级缓存，会抛出如下异常
>
> ```
> Caused by: java.lang.IllegalArgumentException: Caches collection already contains value for com.**.*Mapper
> ```
>
> 这是因为Mapper接口和对应的XML文件是相同的命名空间，想使用二级缓存，两者必须同时配置（如果接口不存在使用注解方式的方法，可以只在XML中配置），因此配置就会出错，这个时候应该使用参照缓存。

###### 参照缓存

Mapper接口配置参照缓存：

```java
@CacheNamespaceRef(*Mapper.class)
```

XML映射文件配置参照缓存：

```xml
<cache-ref namespace="com.**.*Mapper"/>
```



MyBatis中很少会同时使用Mapper接口注解方式和XML映射文件，所以参照缓存并不是为了解决这个问题而设计的。参照缓存除了能够通过引用其他缓存减少配置外，主要的作用是解决脏读。

#### 使用二级缓存

读写缓存：

​	MyBatis使用`SerializedCache(org.apache.ibatis.cache.decorators.SerializedCache)`序列化缓存类来实现可读写缓存，并通过序列化和反序列化来保证通过缓存获取数据时，得到的是一个新的实例。因此使用可读写缓存，可以使用`SerializedCache`序列化缓存。这个缓存类要求所有被序列化的对象必须实现`Serializable(java.io.Serializable)`接口。

只读缓存：

​	MyBatis就会使用`Map`来存储缓存值，这种情况下，从缓存中获取的对象就是同一个实例。



​	MyBatis默认提供的缓存实现是基于`Map`实现的内存缓存，已经可以满足基本的应用。但是当需要缓存大量的数据时，不能仅仅通过提高内存来使用MyBatis的二级缓存，还可以选择一些类似EhCache的缓存框架或Redis缓存数据库等工具来保存Mybatis的二级缓存数据。

#### 集成Redis缓存

[MyBatis Redis integration](http://www.mybatis.org/redis-cache/)

```xml
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-redis</artifactId>
    <version>1.0.0-beta2</version>
</dependency>
```

在`src/main/resources`目录下新增`redis.properties`配置文件

```xml
<cache type="org.mybatis.caches.redis.RedisCache"/>
```

其他集成：

- ignite-cache：https://github.com/mybatis/ignite-cache 
- couchbase-cache：https://github.com/mybatis/couchbase-cache 
- caffeine-cache：https://github.com/mybatis/caffeine-cache
- memcached-cache：https://github.com/mybatis/memcached-cache
- oscache-cache：https://github.com/mybatis/oscache-cache
- redis-cache：https://github.com/mybatis/redis-cache
- ehcache-cache：https://github.com/mybatis/ehcache-cache

#### 脏数据的产生和避免

​	二级缓存虽然能提高应用效率，减轻数据库服务器的压力，但是如果使用不当，很容易产生脏数据。这些脏数据会在不知不觉中影响业务逻辑，影响影响应用的实效，所以我们需要了解在MyBatis缓存中脏数据库是如何产生的，也要掌握避免脏数据的技巧。

​	MyBatis的二级缓存是和命名空间绑定的，所以通常情况下每一个Mapper映射文件都拥有自己的二级缓存，不同Mapper的二级缓存互不影响。

​	由于关系型数据库的设计，使得很多时候需要关联多个表才能获得想要的数据。在关联多表查询时会将该查询放到某个命名空间下的映射文件中，这样一个多表的查询就会缓存在该命名空间的二级缓存中。涉及这些表的增、删、改、操作通常不在一个映射文件中，它们的命名空间不同，因此当有数据变化时，多表查询的缓存未必会被清空，这种情况下就会产生脏数据库。

​	使用参照缓存，可以避免脏数据问题。当某几个表可以作为一个业务整体时，通常是让几个会关联的ER表同时使用同一个二级缓存，这样就能解决脏数据问题。

​	虽然这样可以解决脏数据的问题，但是并不是所有的关联查询都可以这么解决，如果有几十个表甚至所有表都以不同的关联关系存在于各自的映射文件中时，使用参照缓存显然没有意义。

#### 二级缓存适用场景

二级缓存虽然好处很多，但并不是什么时候都可以使用。以下场景中，推荐使用二级缓存：

1. 以查询为主的应用中，只有尽可能少的增、删、改操作。
2. 绝大多数以单表操作存在时，由于很少存在互相关联的情况，因此不会出现脏数据。
3. 可以按业务划分对表进行分组时，如关联的表比较少，可以通过参照缓存进行配置。

除了推荐使用的情况，如果脏读对系统没有影响，也可以考虑使用。在无法保证数据不出现脏读的情况下，建议在业务层使用可控制的缓存代替二级缓存。

##  `<settings>`配置：

- `cacheEnabled`：二级缓存的全局开关，默认值是`true`，初始状态为启用状态。

- `mapUnderscoreToCamelCase`：

- `aggressiveLazyLoading`：

- `lazyLoadTriggerMethods`：当调用配置中的方法时，加载全部的延迟加载数据。默认值为“`equals`”、“`clone`”、“`hashCode`”、“`toString`”。





## XML配置

### 类型处理（typeHandlers）

#### 处理枚举类型

























































































