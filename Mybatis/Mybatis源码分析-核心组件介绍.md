# 1



- Configuration：用于描述Mybatis的主配置信息，其他组件需要获取配置信息时，直接通过Configuration对象获取。除此之外，Mybatis在应用启动时，将Mapper配置信息、类型别名、TypeHandler等注册到Configuration组件中，其他组件需要这些信息时，也可以从Configuration对象中获取。
- MappedStatement：MappedStatement用于描述Mapper中的SQL配置信息，是对Mapper XML配置文件中<select|update|delete|insert>等标签或者@Select/@Update等注解配置信息的封装。
- SqlSeesion：SqlSession是Mybatis提供的面向用户的API，表示和数据库交互时的会话对象。