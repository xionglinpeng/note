

Debian包： https://www.debian.org/distrib/packages

RPM包: http://rpm.pbone.net/



docker账号

docker46251



https://wk4vahax.mirror.aliyuncs.com







在基于XML配置元数据，在Bean的配置信息中我们可以使用`<constructor-arg>`和`<property>`标签来实现Spring的依赖注入。除了这两种方式之外，Spring也可以在不使用`<constructor-arg>`和`<property>`标签的情况下自动装配Bean之间的依赖关系。

使用Bean标签的autowire属性指定具体的自动装配模式，Spring提供了5种自动装配模式：

| 模式        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| no          | 这是默认设置，意味着它没有使用自动装配模式。                 |
| byName      | 由属性名自动装配。Spring容器看到bean采用了自动装配byName模式（autowire="byName"），然后根据它的属性名在Spring容器中寻找与属性名相同bean进行关联。 |
| byType      | 由属性的类型自动装配。Spring容器看到bean采用了自动装配的byType模式（autowire="byType"），然后根据属性类型在Spring容器中寻找与属性类型相同的bean进行装配，如果有多个bean都匹配，将抛出异常。 |
| constructor | 类似byType，但该类型适用于构造参数类型。如果在容器中没有一个与构造函数参数类型匹配的bean，将抛出异常。 |
| autodetect  | Spring首先尝试通过constructor使用自动装配来连接，如果他不执行，Spring尝试通过byType来自动装配。 |





在使用Spring依赖注入的时候，我们常常希望有些依赖Bean必须被注入初始化，为此，Spring 2.5.x版本在XML配置上提供了一个属性`denpendency-check`属性进行依赖检查，如果检查不通过，将抛出异常。

*示例：denpendency-check*

```xml

```

`denpendency-check`属性有四个可选值：

- `none`：默认值，不进行任何检查。
- `simple`：只检查简单类型属性以及集合类型属性。
- `object`：检查除简单类型属性以及集合类型属性外的引用类型属性。
- `all`：检查所有类型属性。

当检查没有通过时，会抛出异常``。

> 这个属性仅仅存在于Spring2.x.x版本中，从3.0.0.RELEASE版本开始，已经被废弃（虽然被废弃，但相关源码还存在）。
>
> 既然Spring废弃了这个属性的功能，那么一定有替代它的功能出现：
>
> 1. Use constructors (constructor injection instead of setter injection) exclusively to ensure the right properties are set.
> 2. Create setters with a dedicated init method implemented.
> 3. Create setters with @Required annotation when the property is required.
> 4. Use @Autowired-driven injection which also implies a required property by default.
>
> 中文翻译
>
> 1. 使用构造方法（使用构造方法代替setter注入）专门用来确认特定属性被初始化。
>
> 2. 用init方法初始化setter的属性。
>
> 3. 在需要强制进行初始化的setters上标注@Required。
>
>    参考资料：https://www.mkyong.com/spring/spring-dependency-checking-with-required-annotation/
>
> 4. 使用@Autowired-drivern，默认情况下也意味着是必须得属性。





