# SqlSession执行Mapper过程

Mapper由两部分组成：

1. Mapper接口。
2. 通过注解或者XML文件配置的SQL语句。

## 一、Mapper接口的注册过程



## 二、MappedStatement注册过程

Mybatis通过MappedStatement类型描述Mapper的SQL配置信息。

SQL配置有两种方式：

1. XML文件配置。
2. Java注解配置（本质就是一种轻量级的配置信息）。

在Configuration类中有一个mappedStatements属性，该属性用于注册Mybatis中所有的MappedStatement对象。

相关代码如下：

```java
protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection")
    .conflictMessageProducer((savedValue, targetValue) ->
                             ". please check " + savedValue.getResource() + " and " + targetValue.getResource());
```

mappedStatements是一个Map对象，它的key为Mapper SQL配置的id。

- 如果SQL是通过XML配置的，则id为命令空间加上`<select|update|delete|insert>`标签的id。
- 如果SQL是通过Java注解配置的，则id为Mapper接口的完全限定命令+方法名称。

在Configuration中与mappedStatements相关的方法如下：

```shell

```



在之前已经说明了Mybatis是通过`XMLConfigBuilder`来解析主配置文件，而在主配置文件中是通过`<mappers>`标签来配置Mapper配置文件的。所以在`XMLConfigBuilder`中是通过`mapperElement(XNode)`方法来解析的，相关代码如下：

```java
private void parseConfiguration(XNode root) {
    try {
        //something ...
        mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
}
```

mapperElement方法实现：

```java
private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            if ("package".equals(child.getName())) {
                String mapperPackage = child.getStringAttribute("name");
                configuration.addMappers(mapperPackage);
            } else {
                String resource = child.getStringAttribute("resource");
                String url = child.getStringAttribute("url");
                String mapperClass = child.getStringAttribute("class");
                if (resource != null && url == null && mapperClass == null) {
                    ErrorContext.instance().resource(resource);
                    InputStream inputStream = Resources.getResourceAsStream(resource);
                    XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
                    mapperParser.parse();
                } else if (resource == null && url != null && mapperClass == null) {
                    ErrorContext.instance().resource(url);
                    InputStream inputStream = Resources.getUrlAsStream(url);
                    XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
                    mapperParser.parse();
                } else if (resource == null && url == null && mapperClass != null) {
                    Class<?> mapperInterface = Resources.classForName(mapperClass);
                    configuration.addMapper(mapperInterface);
                } else {
                    throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
                }
            }
        }
    }
}
```

上述代码中，首先获取`<mappers>`的所有子标签，然后根据不同的标签做不同的处理。

`<mappers>`标签配置Mapper信息有以下四种方式：

```xml
<mappers>
    <!--通过resource属性指定Mapper文件的classpath路径-->
    <mapper resource="com/mybatis/mapper/userMapper.xml"/>
    <!--通过class属性指定Mapper接口的完全限定名-->
    <mapper class="com.mybatis.mapper.UserMapper"/>
    <!--通过url属性指定Mapper文件的网络路径-->
    <mapper url="file:///var/mappers/userMapper.xml"/>
    <!--通过package标签指定Mapper接口所在的包名-->
    <package name="com.mybatis.mapper"/>
</mappers>
```

针对这四种情况，`mapperElement(XNode)`方法都做了不同的处理

- package

- class

- resource

- url

Mybatis SQL配置文件的解析是由`org.apache.ibatis.builder.xml.XMLMapperBuilder`类去处理的，上述代码中`resource`和`url`虽然来源不同，但他们都是SQL XML配置文件。

首先创建了一个XMLMapperBuilder对象，然后调用XMLMapperBuilder对象`parse()`方法解析，其相关对象数据已通过构造方法传入，即每一个SQL配置都有一个对应的XMLMapperBuilder对象去解析。

  *`parse()`方法代码如下：*

```java
public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
        //调用XPathParser的evalNode()方法获取根节点对应的XNode对象
        configurationElement(parser.evalNode("/mapper"));
        //将资源路径添加到Configuration对象中
        configuration.addLoadedResource(resource);
        bindMapperForNamespace();
    }

    parsePendingResultMaps();
    parsePendingCacheRefs();
    parsePendingStatements();
}
```

上面的代码中，调用XPathParser的evalNode()方法获取根节点对应的XNode对象，然后configurationElement()方法做进一步解析。

*`configurationElement()`方法代码如下：*

```java
private void configurationElement(XNode context) {
    try {
        //获取命名空间
        String namespace = context.getStringAttribute("namespace");
        if (namespace == null || namespace.isEmpty()) {
            throw new BuilderException("Mapper's namespace cannot be empty");
        }
        //设置当前正在解析的Mapper配置的命名空间
        builderAssistant.setCurrentNamespace(namespace);
        //解析<cache-ref>标签
        cacheRefElement(context.evalNode("cache-ref"));
        //解析<cache>标签
        cacheElement(context.evalNode("cache"));
        //解析所有的<parameterMap>标签
        parameterMapElement(context.evalNodes("/mapper/parameterMap"));
        //解析所有的<resultMap>标签
        resultMapElements(context.evalNodes("/mapper/resultMap"));
        //解析所有的<sql>标签
        sqlElement(context.evalNodes("/mapper/sql"));
        //解析所有的<select|insert|update|delete>标签
        buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
    }
}
```

如上面代码所示，`configurationElement()`方法最Mapper SQL配置文件中的所有标签进行解析。



**`<select|insert|update|delete>`标签**

`buildStatementFromContext()`方法是对`<select|insert|update|delete>`标签进行解析，但是它没有直接进行解析，而是再次交由`org.apache.ibatis.builder.xml.XMLStatementBuilder`类型进行解析。

*`buildStatementFromContext()`方法代码如下：*

```java
private void buildStatementFromContext(List<XNode> list) {
    if (configuration.getDatabaseId() != null) {
        buildStatementFromContext(list, configuration.getDatabaseId());
    }
    buildStatementFromContext(list, null);
}

private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    for (XNode context : list) {
        //通过XMLStatementBuilder对象对<select|insert|update|delete>标签进行解析
        final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
        try {
            //调用parseStatementNode()方法进行解析
            statementParser.parseStatementNode();
        } catch (IncompleteElementException e) {
            configuration.addIncompleteStatement(statementParser);
        }
    }
}
```

如上面代码所示，在`buildStatementFromContext()`方法中对所有的`<select|insert|update|delete>`标签的XNode对象进行迭代，并为每一个XNode对象创建对应的XMLStatementBuilder对象，然后调用XMLStatementBuilder对象的`parseStatementNode()`方法进行解析。

*`parseStatementNode()`方法代码如下：*

```java
public void parseStatementNode() {
    String id = context.getStringAttribute("id");
    String databaseId = context.getStringAttribute("databaseId");

    if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
        return;
    }

    String nodeName = context.getNode().getNodeName();
    SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
    boolean useCache = context.getBooleanAttribute("useCache", isSelect);
    boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);

    // Include Fragments before parsing
    XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
    includeParser.applyIncludes(context.getNode());

    String parameterType = context.getStringAttribute("parameterType");
    Class<?> parameterTypeClass = resolveClass(parameterType);

    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);

    // Parse selectKey after includes and remove them.
    processSelectKeyNodes(id, parameterTypeClass, langDriver);

    // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
    KeyGenerator keyGenerator;
    String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
    keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
    if (configuration.hasKeyGenerator(keyStatementId)) {
        keyGenerator = configuration.getKeyGenerator(keyStatementId);
    } else {
        keyGenerator = context.getBooleanAttribute("useGeneratedKeys",
                                                   configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))
            ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
    }

    SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
    StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));
    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    String resultType = context.getStringAttribute("resultType");
    Class<?> resultTypeClass = resolveClass(resultType);
    String resultMap = context.getStringAttribute("resultMap");
    String resultSetType = context.getStringAttribute("resultSetType");
    ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);
    if (resultSetTypeEnum == null) {
        resultSetTypeEnum = configuration.getDefaultResultSetType();
    }
    String keyProperty = context.getStringAttribute("keyProperty");
    String keyColumn = context.getStringAttribute("keyColumn");
    String resultSets = context.getStringAttribute("resultSets");

    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
                                        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
                                        resultSetTypeEnum, flushCache, useCache, resultOrdered,
                                        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
}
```

## 三、Mapper方法调用过程详解

