# Class类文件结构





## 1、魔数

## 2、常量池



## 3、访问标志

## 4、类索引、父类索引与接口索引集合





## 5、字段表集合

字段表（field_info）用于描述接口或者类中声明的变量。Java语言中的“字段”（Field）包括类级别变量以及实例级别变量，但不包括在方法内部声明的局部变量。

字段可以包括的修饰符：

- 字段的作用域：public、private、protected修饰符
- 是实例变量还是类变量：static修饰符
- 可变性：final修饰符
- 并发可见性：volatile修饰符，是否强制从主内存读写
- 可否被序列化：transient修饰符
- 字段数据类型：基本类型、对象、数组
- 字段名称

> 上述修饰符中，除了字段数据类型和字段名称之外，其他都是布尔类型，使用标志位来表示。

*字段表结构：*

```
field_info {
    u2 access_flags;
    u2 name_index;
    u2 descriptor_index;
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```

结构体field_info的项目如下：

- access_flags：字段的修饰符，与类中的项目access_flags类似。

  *可以设置的标志位和含义如下表所示：*

  | flag name       | flag value | 含义                     |
  | --------------- | ---------- | ------------------------ |
  | `ACC_PUBLIC`    | `0x0001`   | 字段是否public           |
  | `ACC_PRIVATE`   | `0x0002`   | 字段是否private          |
  | `ACC_PROTECTED` | `0x0004`   | 字段是否protected        |
  | `ACC_STATIC`    | `0x0008`   | 字段是否static           |
  | `ACC_FINAL`     | `0x0010`   | 字段是否final            |
  | `ACC_VOLATILE`  | `0x0040`   | 字段是否volatile         |
  | `ACC_TRANSIENT` | `0x0080`   | 字段是否transient        |
  | `ACC_SYNTHETIC` | `0x1000`   | 字段是否由编译器自动产生 |
  | `ACC_ENUM`      | `0x4000`   | 字段是否`enum`           |

  Note：上述表格中列出的flag value仅仅是二进制的16进制表示，因此`ACC_PRIVATE`的二进制表示为`00000000 00000010`，`ACC_STATIC`的二进制表示为`00000000 00001000`，如果字段声明为`private static`，则二进制表示为`00000000 00001010`，对应的十六进制为`0x000A`。所以，在解析Class文件的时候可能会发现对应字段的flag value为`0x0009`、`0x000A`等，与上述表格中的flag value没有匹配的情况。以此类推，其他组合也是一样的。

- name_index：对常量池项的引用，代表字段的简单名称。

- descriptor_index：对常量池项的引用，代表字段的描述符。

- attributes_count：字段属性表数量。

- attributes[attributes_count]：



字段表集合中不会列出父类或父接口中继承而来的字段，但有可能出现原本Java代码之中不存在的字段，譬如在内部类中为了保持对外部类的访问性，编译器就会自动添加指向外部类实例的字段。另外，在Java语言中字段是无法重载的，两个字段的数据类型，修饰符不管是否相同，都必须使用不一样的名称，但是对于Class文件格式来讲，只要两个字段的描述符不是完全相同，那字段重名就是合法的。



## 6、方法表集合



方法表结构

```
method_info {
    u2 access_flags;
    u2 name_index;
    u2 descriptor_index;
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```

结构体method_info的项目如下：

- access_flags：方法的修饰符，与类中的项目access_flags类似。

  *可以设置的标志位和含义如下表所示：*

- | flag name          | flag value | 含义                           |
  | ------------------ | ---------- | ------------------------------ |
  | `ACC_PUBLIC`       | `0x0001`   | 方法是否为public               |
  | `ACC_PRIVATE`      | `0x0002`   | 方法是否为private              |
  | `ACC_PROTEDTED`    | `0x0004`   | 方法是否为protected            |
  | `ACC_STATIC`       | `0x0008`   | 方法是否为static               |
  | `ACC_FINAL`        | `0x0010`   | 方法是否为final                |
  | `ACC_SYNCHRONIZED` | `0x0020`   | 方法是否为synchronized         |
  | `ACC_BRIDGE`       | `0x0040`   | 方法是否由编译器产生的桥接方法 |
  | `ACC_VARARGS`      | `0x0080`   | 方法是否接受不定参数           |
  | `ACC_NATIVE`       | `0x0100`   | 方法是否为native               |
  | `ACC_ABSTRACT`     | `0x0400`   | 方法是否为abstract             |
  | `ACC_STRICTFP`     | `0x0800`   | 方法是否为strictfp             |
  | `ACC_SYNTHETIC`    | `0x1000`   | 方法是否由编译器自动产生       |

- name_index：对常量池项的引用，代表方法的简单名称。

- descriptor_index：对常量池项的引用，代表方法的描述符。

- attributes_count：方法属性表数量。

- attributes[attributes_count]：

## 7、属性（Attributes）





### 7.1、code属性



```
Code_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 max_stack;
    u2 max_locals;
    u4 code_length;
    u1 code[code_length];
    u2 exception_table_length;
    { 	u2 start_pc;
        u2 end_pc;
        u2 handler_pc;
        u2 catch_type;
    }exception_table[exception_table_length];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```



- `attribute_name_index`：一项指向`CONSTANT_Utf8_info`型常量的索引，此常量固定值为“Code”，它代表了该属性的属性名称。
- `attribute_length`：属性值的长度，由于`attribute_name_index`和`attribute_length`一共为6个字节，所以属性值的长度固定为整个属性表长度减去6个字节。
- `max_stack`：操作数栈深度的最大值。
- `max_locals`：局部变量表所属的存储空间。
- `code_length`：字节码长度。
- `code[code_length]`：字节码指令
- `exception_table[exception_table_length]`：异常处理表
  - `start_pc`：
  - `end_pc`：
  - `handler_pc`：
  - `catch_type`：
- `attributes_count`：
- `attributes[attributes_count]`：

### 





