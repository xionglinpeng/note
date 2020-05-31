# SqlSession的创建过程

## XPath方式解析XML文件





## Configuration实例创建过程



- `<properties>`：用于配置属性信息，这些属性的值可以通过`${...}`方式引用。
- `<settings>`：通过一些属性来控制Mybatis运行时的一些行为。
- `<typeAliases>`：用于配置类型别名，目的是为Java类型设置一个更短的名字。
- `<plugins>`：用于注册用户指定的插件信息。
- `<objectFactory>`：Mybatis通过对象工厂（ObjectFactory）创建参数对象和结果集映射对象，默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。
- `<objectWrapperFactory>`：
- `<reflectorFactory>`：
- `<environments>`：
- `<databaseIdProvider>`：
- `<typeHandlers>`：
- `<mappers>`：

