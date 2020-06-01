# SqlSession的创建过程

## XPath方式解析XML文件





## Configuration实例创建过程

Mybatis是通过`org.apache.ibatis.builder.xml.XMLConfigBuilder`类来完成Configuration对象的创建工作。

*测试代码如下：*

```java
@Test
public void testConfiguration() throws IOException {
    Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
    //创建XMLConfigBuilder实例
    XMLConfigBuilder builder = new XMLConfigBuilder(reader);
    //调用XMLConfigBuilder.parse()方法，解析XML创建Configuration对象
    Configuration configuration = builder.parse();
}
```





- `<properties>`：用于配置属性信息，这些属性的值可以通过`${...}`方式引用。
- `<settings>`：通过一些属性来控制Mybatis运行时的一些行为。
- `<typeAliases>`：用于配置类型别名，目的是为Java类型设置一个更短的名字。
- `<plugins>`：用于注册用户指定的插件信息。
- `<objectFactory>`：Mybatis通过对象工厂（ObjectFactory）创建参数对象和结果集映射对象，默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。
- `<objectWrapperFactory>`：Mybatis通过ObjectWrapperFactory创建ObjectWrapper对象，通过ObjectWrapper对象能够很方便地获取对象的属性、方法名等反射信息。`<objectWrapperFactory>`标签用于配置用户自定义的ObjectWrapperFactory。
- `<reflectorFactory>`：Mybatis反射工程（ReflectorFactory）创建描述Java类型反射信息的Reflector对象，通过Reflector对象能够很方便地获取Class对象的Setter/Getter方法、属性等信息。`<reflectorFactory>`标签用于配置自定义的反射工厂。
- `<environments>`：用于配置Mybatis数据连接相关的环境及事务管理器信息。通过该标签可以配置多个环境信息，然后指定具体使用哪个。
- `<databaseIdProvider>`：Mybatis能够根据不同的数据库厂商执行不同的SQL语句，该标签用于配置数据库厂商的信息。
- `<typeHandlers>`：用于注册用户自定义的类型处理器（TypeHandler）。
- `<mappers>`：用于配置Mybatis Mapper信息。

