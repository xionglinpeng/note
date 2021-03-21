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



| 字节码 | 助记符      | 指令含义                                                     |
| ------ | ----------- | ------------------------------------------------------------ |
| 0x00   | nop         | 什么都不做                                                   |
| 0x01   | aconst_null | 将null推送至栈顶                                             |
| 0x02   | iconst_m1   | 将int型-1推送至栈顶                                          |
| 0x03   | iconst_0    | 将int型0推送至栈顶                                           |
| 0x04   | iconst_1    | 将int型1推送至栈顶                                           |
| 0x05   | iconst_2    | 将int型2推送至栈顶                                           |
| 0x06   | iconst_3    | 将int型3推送至栈顶                                           |
| 0x07   | iconst_4    | 将int型4推送至栈顶                                           |
| 0x08   | iconst_5    | 将int型5推送至栈顶                                           |
| 0x09   | lconst_0    | 将long型0推送至栈顶                                          |
| 0x0a   | lconst_1    | 将long型1推送至栈顶                                          |
| 0x0b   | fconst_0    | 将float型0推送至栈顶                                         |
| 0x0c   | fconst_1    | 将float型1推送至栈顶                                         |
| 0x0d   | fconst_2    | 将float型2推送至栈顶                                         |
| 0x0e   | dconst_0    | 将double型0推送至栈顶                                        |
| 0x0f   | dconst_1    | 将double型1推送至栈顶                                        |
| 0x10   | bipush      | 将单字节的常量值（-128~127）推送至栈顶                       |
| 0x11   | sipush      | 将一个短整型常量值（-32768~32767）推送至栈顶                 |
| 0x12   | ldc         | 将int、float或String型常量值从常量池中推送至栈顶             |
| 0x13   | ldc_w       | 将int、float或String型常量值从常量池中推送至栈顶（宽索引）   |
| 0x14   | ldc2_w      | 将long或double型常量值从常量池中推送至栈顶（宽索引）         |
| 0x15   | iload       | 将指定的int型本地变量推送至栈顶                              |
| 0x16   | lload       | 将指定的long型本地变量推送至栈顶                             |
| 0x17   | fload       | 将指定的float型本地变量推送至栈顶                            |
| 0x18   | dload       | 将指定的double型本地变量推送至栈顶                           |
| 0x19   | aload       | 将指定的引用类型本地变量推送至栈顶                           |
| 0x1a   | iload_0     | 将第一个int型本地变量推送至栈顶                              |
| 0x1b   | iload_1     | 将第二个int型本地变量推送至栈顶                              |
| 0x1c   | iload_2     | 将第三个int型本地变量推送至栈顶                              |
| 0x1d   | iload_3     | 将第四个int型本地变量推送至栈顶                              |
| 0x1e   | lload_0     | 将第一个long型本地变量推送至栈顶                             |
| 0x1f   | lload_1     | 将第二个long型本地变量推送至栈顶                             |
| 0x20   | lload_2     | 将第三个long型本地变量推送至栈顶                             |
| 0x21   | lload_3     | 将第四个long型本地变量推送至栈顶                             |
| 0x22   | fload_0     | 将第一个float型本地变量推送至栈顶                            |
| 0x23   | fload_1     | 将第二个float型本地变量推送至栈顶                            |
| 0x24   | fload_2     | 将第三个float型本地变量推送至栈顶                            |
| 0x25   | fload_3     | 将第四个float型本地变量推送至栈顶                            |
| 0x26   | dload_0     | 将第一个double型本地变量推送至栈顶                           |
| 0x27   | dload_1     | 将第二个double型本地变量推送至栈顶                           |
| 0x28   | dload_2     | 将第三个double型本地变量推送至栈顶                           |
| 0x29   | dload_3     | 将第四个double型本地变量推送至栈顶                           |
| 0x2a   | aload_0     | 将第一个引用类型本地变量推送至栈顶                           |
| 0x2b   | aload_1     | 将第二个引用类型本地变量推送至栈顶                           |
| 0x2c   | aload_2     | 将第三个引用类型本地变量推送至栈顶                           |
| 0x2d   | aload_3     | 将第四个引用类型本地变量推送至栈顶                           |
| 0x2e   | iaload      | 将int型数组指定索引的值推送至栈顶                            |
| 0x2f   | laload      | 将long型数组指定索引的值推送至栈顶                           |
| 0x30   | faload      | 将float型数组指定索引的值推送至栈顶                          |
| 0x31   | daload      | 将double型数组指定索引的值推送至栈顶                         |
| 0x32   | aaload      | 将引用类型数组指定索引的值推送至栈顶                         |
| 0x33   | baload      | 将boolean或byte型数组指定索引的值推送至栈顶                  |
| 0x34   | caload      | 将char型数组指定索引的值推送至栈顶                           |
| 0x35   | saload      | 将short型数组指定索引的值推送至栈顶                          |
| 0x36   | istore      | 将栈顶int型数值存入指定本地变量                              |
| 0x37   | lstore      | 将栈顶long型数值存入指定本地变量                             |
| 0x38   | fstore      | 将栈顶float型数值存入指定本地变量                            |
| 0x39   | dstore      | 将栈顶double型数值存入指定本地变量                           |
| 0x3a   | astore      | 将栈顶引用型数值存入指定本地变量                             |
| 0x3b   | istore_0    | 将栈顶int型数值存入第一个本地变量                            |
| 0x3c   | istore_1    | 将栈顶int型数值存入第二个本地变量                            |
| 0x3d   | istore_2    | 将栈顶int型数值存入第三个本地变量                            |
| 0x3e   | istore_3    | 将栈顶int型数值存入第四个本地变量                            |
| 0x3f   | lstore_0    | 将栈顶long型数值存入第一个本地变量                           |
| 0x40   | lstore_1    | 将栈顶long型数值存入第二个本地变量                           |
| 0x41   | lstore_2    | 将栈顶long型数值存入第三个本地变量                           |
| 0x42   | lstore_3    | 将栈顶long型数值存入第四个本地变量                           |
| 0x43   | fstore_0    | 将栈顶float型数值存入第一个本地变量                          |
| 0x44   | fstore_1    | 将栈顶float型数值存入第二个本地变量                          |
| 0x45   | fstore_2    | 将栈顶float型数值存入第三个本地变量                          |
| 0x46   | fstore_3    | 将栈顶float型数值存入第四个本地变量                          |
| 0x47   | dstore_0    | 将栈顶double型数值存入第一个本地变量                         |
| 0x48   | dstore_1    | 将栈顶double型数值存入第二个本地变量                         |
| 0x49   | dstore_2    | 将栈顶double型数值存入第三个本地变量                         |
| 0x4a   | dstore_3    | 将栈顶double型数值存入第四个本地变量                         |
| 0x4b   | store_0     | 将栈顶引用型数值存入第一个本地变量                           |
| 0x4c   | store_1     | 将栈顶引用型数值存入第二个本地变量                           |
| 0x4d   | store_2     | 将栈顶引用型数值存入第三个本地变量                           |
| 0x4e   | store_3     | 将栈顶引用型数值存入第四个本地变量                           |
| 0x4f   | iastore     | 将栈顶int型数值存入指定数组的指定索引位置                    |
| 0x50   | lastore     | 将栈顶long型数值存入指定数组的指定索引位置                   |
| 0x51   | fastore     | 将栈顶float型数值存入指定数组的指定索引位置                  |
| 0x52   | dastore     | 将栈顶double型数值存入指定数组的指定索引位置                 |
| 0x53   | aastore     | 将栈顶引用型数值存入指定数组的指定索引位置                   |
| 0x54   | bastore     | 将栈顶boolean或byte型数值存入指定数组的指定索引位置          |
| 0x55   | castore     | 将栈顶char型数值存入指定数组的指定索引位置                   |
| 0x56   | sastore     | 将栈顶short型数值存入指定数组的指定索引位置                  |
| 0x57   | pop         | 将栈顶数值弹出（数值不能是long或double类型的）               |
| 0x58   | pop2        | 将栈顶的一个（对于long或double类型）或两个数值（对于非long或double类型）弹出 |
| 0x59   | dup         | 复制栈顶数值并将复制值压入栈顶                               |
| 0x5a   | dup_x1      | 复制栈顶数值并将两个复制值压入栈顶                           |
| 0x5b   | dup_x2      | 复制栈顶数值并将三个（或两个）复制值压入栈顶                 |
| 0x5c   | dup2        | 复制栈顶一个（对于long或double类型）或两个（对于非long或double类型）数值并将复制值压入栈顶 |
| 0x5d   | dup2_x1     | dup_x1指令的双倍版本                                         |
| 0x5e   | dup2_x2     | dup_x2指令的双倍版本                                         |
| 0x5f   | swap        | 将栈最顶端的两个数值互换（数值不能是long或double类型）       |
| 0x60   | iadd        | 将栈顶两int型数值相加并将结果压入栈顶                        |
| 0x61   | ladd        | 将栈顶两long型数值相加并将结果压入栈顶                       |
| 0x62   | fadd        | 将栈顶两float型数值相加并将结果压入栈顶                      |
| 0x63   | dadd        | 将栈顶两double型数值相加并将结果压入栈顶                     |
| 0x64   | isub        | 将栈顶两int型数值相减并将结果压入栈顶                        |
| 0x65   | lsub        | 将栈顶两long型数值相减并将结果压入栈顶                       |
| 0x66   | fsub        | 将栈顶两float型数值相减并将结果压入栈顶                      |
| 0x67   | dsub        | 将栈顶两double型数值相减并将结果压入栈顶                     |
| 0x68   | imul        | 将栈顶两int型数值相乘并将结果压入栈顶                        |
| 0x69   | lmul        | 将栈顶两long型数值相乘并将结果压入栈顶                       |
| 0x6a   | fmul        | 将栈顶两float型数值相乘并将结果压入栈顶                      |
| 0x6b   | dmul        | 将栈顶两double型数值相乘并将结果压入栈顶                     |
| 0x6c   | idiv        | 将栈顶两int型数值相除并将结果压入栈顶                        |
| 0x6d   | ldiv        | 将栈顶两long型数值相除并将结果压入栈顶                       |
| 0x6e   | fdiv        | 将栈顶两float型数值相除并将结果压入栈顶                      |
| 0x6f   | ddiv        | 将栈顶两double型数值相除并将结果压入栈顶                     |
| 0x70   | irem        | 将栈顶两int型数值进行取模运算并将结果压入栈顶                |
| 0x71   | lrem        | 将栈顶两long型数值进行取模运算并将结果压入栈顶               |
| 0x72   | frem        | 将栈顶两float型数值进行取模运算并将结果压入栈顶              |
| 0x73   | drem        | 将栈顶两double型数值进行取模运算并将结果压入栈顶             |
| 0x74   | ineg        | 将栈顶int型数值取负并将结果压入栈顶                          |
| 0x75   | lneg        | 将栈顶long型数值取负并将结果压入栈顶                         |
| 0x76   | fneg        | 将栈顶float型数值取负并将结果压入栈顶                        |
| 0x77   | dneg        | 将栈顶double型数值取负并将结果压入栈顶                       |
| 0x78   | ishl        | 将int型数值左移指定位数并将结果压入栈顶                      |
| 0x79   | lshl        | 将long型数值左移指定位数并将结果压入栈顶                     |
| 0x7a   | ishr        | 将int型数值右（带符号）移指定位数并将结果压入栈顶            |
| 0x7b   | lshr        | 将long型数值右（带符号）移指定位数并将结果压入栈顶           |
| 0x7c   | iushr       | 将int型数值右（无符号）移指定位数并将结果压入栈顶            |
| 0x7d   | lushr       | 将long型数值右（无符号）移指定位数并将结果压入栈顶           |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |
|        |             |                                                              |



