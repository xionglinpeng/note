# VUE



指令

v-if

v-for

v-bind



v-show



## Class与Style绑定



v-bind:class

v-bind:style





## 组件





```html
<script type="text/x-template"></script>

模板必须放在vue.js之前，但是不能放在Root VUE对象的DOM里面。否则，在初始加载的时候Root vue就会将模板当做DOM中的一个节点去加载，如果模板中还有属于组件的{{}}表达式，那么将不能解析。
错误：


正确：

```

### 组件注册

### 组件通信

### 组件实例访问

#### 父组件访问子组件



> ref ：v-if等标签会使ref失效
>
> Thymealf的`th:ref`标签跟vue的`ref`标签冲突，即Thymealf的`th:ref`标签有它特殊的语义，解决方案是：`th:attr="ref=|string${Expression}|"`



#### 子组件访问父组件













