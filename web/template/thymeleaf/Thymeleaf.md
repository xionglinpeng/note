# Thymeleaf

```html
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
```









$

#

@

*

~



th:each

```
th:each="item : ${items}"
```





## 表达式内字符串拼接

表达式内字符串拼接的字符串拼接方式有三种方式：

- 表达式内部加号（+）拼接

  ```html
  <p th:text="${'hello'+dto.str}"></p>
  ```

  > 其实`${}`本质上是SPEL表达式：
  >
  > 当我们的表达式就是需要表示一个字符串时应该如何表示呢？这个时候需要通过单引号“`’`”来进行包裹。而当我们的字符串中包含单引号时，那么对应的单引号需要使用一个单引号进行转义，即连续两个单引号。
  >
  > 例如，`'abc''def'`，渲染出来之后就是`abc'def`。
  >
  > 示例：
  >
  > ```html
  > th:onclick="@{${'location.href='''+dto.link+''''}}"
  > ```



- 表达式外部加号（+）拼接

  ```html
  <p th:text="'hello'+${dto.str}"></p>
  ```

  

- 表达式外部竖线（|）拼接

  ```html
  <p th:text="|hello${dto.str}|"></p>
  ```

  > 没有表达式表达式内部部竖线（|）拼接。





Thymeleaf插件



Thymeleaf警告



