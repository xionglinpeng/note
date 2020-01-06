# ImportSelector



## DeferredImportSelector

`ImportSelector`的一种扩展，在处理完所有`@Configuration`类型的Bean之后运行。当所选导入为`@Conditional`时，这种类型的选择器特别有用。

实现类还可以扩展`Ordered`接口，或者使用`@Order`注解来指示相对于其他`DeferredImportSelector`的优先级。

实现类也可以提供导入组，该导入组可以提供跨不同选择器的其他排序和筛选逻辑。



由此我们可以知道，`DeferredImportSelector`的执行时机，是在@Configuration注解中的其他逻辑被处理完毕之后（包括`@ImportResource`、`@Beean`这些注解的处理）再执行，换句话说，`DeferredImportSelector`的执行时机比`ImportSelector`更晚。







