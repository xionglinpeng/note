## 引用

http://c.biancheng.net/cpp/shell/

## 第一个shell脚本

打开文本编辑器，新建一个文件，扩展名为sh（sh代表shell），扩展名并不影响脚本执行，见名知意就好，如果你用php些shell脚本，扩展名就就用php好了。

输入一些代码：

```shell
#!/bin/bash
echo "Hello World!"
```

"`#!`"是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell。echo命令用于向窗口输出文件。

运行Shell脚本有两种方法。

1. 作为可执行程序

   将上面的代码保存为test.sh，并cd到相应的目录：

   ```shell
   chmod +x ./test.sh #使脚本具有执行权限
   ./test.sh #执行脚本
   ```

   注意，一定要写成`./test.sh`，而不是`test.sh`。运行其他二进制程序也一样，直接写`test.sh`，Linux系统会去`PATH`里寻找有没有叫`test.sh`的，而只有`/bin`、`/sbin`、`/usr/bin`、`/usr/sbin`等在`PATH`里，你的当前目录通常不在`PATH`里，所以写成`test.sh`是会找不到命令的，要用`./test.sh`告诉系统，就在当前目录找。

2. 作为解释器参数

   这种运行方式是，直接运行解释器，其参数就是shell脚本的文件名，如：

   ```shell
   /bin/sh test.sh
   /bin/php test.php
   ```

   这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。

再看一个例子，下面的脚本使用read命令从stdin获取输入并赋值给PERSON变量，最后在stdout上输出：

```shell
#!/bin/bash
echo "What is your name?"
read PERSON
echo "Hello, $PERSON"
```

运行脚本

```shell
$ chmod +x ./test.sh
$ ./test.sh
What is your name?
张三
Hello, 张三
```

## Shell变量

变量是任何一种编程语言都必不可少的组成部分，变量用来存放各种数据。脚本语言在定义变量时通常不需要指明类型，直接赋值即可，Shell变量也遵循这个规则。

在Bash shell中，每一个变量的值都是字符串，无论你给变量赋值时有没有使用引号，，值都会以字符串的形式存储。

这意味着，Bash shell在默认情况下不会区分变量类型，即使你将整数和小数赋值给变量，它们也会被视为字符串，这一点和大部分的编程语言不通。

> 例如在java中，变量分为整数、小数、字符串、布尔等多种类型。

当然，如果有必要，你也可以使用`declare`关键字显示定义变量的类型，但在一般情况下没有这个需求，Shell开发者在编写代码时自行注意值的类型即可。

### 定义变量

Shell支持一下三种定义变量的方式：

```shell
variavle=value
variable='value'
variavle="value"
```

variable是变量名，value是赋值给变量的值，**如果value不包含任何空白符（例如空格、Tab缩进等），那么可以不使用引号；如果value包含了空白符，那么就必须使用引号包围起来**。使用单引号和使用双引号也是有区别的，稍后我们会详细说明。

**注意：赋值号的周围不能有空格**，这可能和你熟悉的大部分编程语言都不一样。

Shell变量的命名规范和大部分编程语言都一样：

- 变量名由数字、字母、下划线组成；
- 必须以字母或者下划线开头；
- 不能使用Shell里的关键字（通过help命令可以查看保留关键字）；

变量定义举例：

```shell
url=http://www.baidu.com
echo $url
name='Shell中文网'
echo $name
author="xlp"
echo $author
```

### 使用变量

使用一个定义过的变量，只要在变量名前面加美元符号`$`即可，如：

```shell
author="xlp"
echo $author
echo ${author}
```

变量名外面的花括号`{}`是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，比如下面这种情况：

```shell
skill="Java"
echo "I am good at ${skill}Script"
```

如果不给skill变量加花括号，写成`echo "I am good at $skillScript"`，解释器就会把`$skillScript`当成一个变量（其值为空），代码执行结果就不是我们期望的样子了。

**推荐给所有变量加上或括号`{}`，这是个良好的编程习惯。**

### 修改变量的值

已定义的变量，可以被重新赋值，如：

```shell
url="https://www.baidu.com"
echo ${url}
url="https://www.google.com"
echio ${url}
```

第二次对变量赋值时不能再变量名前加`$`，只有在使用变量时才能加`$`。

### 单引号和双引号的区别

前面我们还留下一个疑问，定义变量时，变量的值可以由单引号`‘’`包围，也可以由双引号`“”`包围，它们到底有什么区别呢？不妨一下面的代码为例来金说明：

```shell
#!/bin/bash
url="https://www.baidu.com"
website1='shell中文网：${url}'
website2="shell中文的：${url}"
echo $website1
echo $website2
```

运行结果：

```txt
shell中文网：${url}
shell中文的：https://www.baidu.com
```

以单引号`‘’`包围变量的值时，单引号里面是什么就输出什么，即是内容中有变量和命令（命令需要反引起来）也会把它们原样输出。这种方式比较适合定义显示纯字符的情况。即不希望解析变量、命令等场景。

以双引号`“”`包围变量的值时，输出时会先解析里面的变量和命令，而不是把双引号中的变量和命令原样输出。这种方式比较适合字符串中附带有变量的和命令并且向将其解析后再输出的变量定义。

**建议：如果变量的内容是数字，那么可以不加引号；如果真的需要原样输出就加单引号；其他没有特别要求的字符串等最好都加上双引号，定义变量时加双引号是最常见的使用场景。**

### 将命令的结果赋值给变量

Shell也支持将命令的执行结果赋值给变量，常见的有一下两种方式：

```shell
variable=`command`
variable=$(command)
```

第一种方式把命令用反引号包围起来，单引号和单引号非常类似，容易产生混淆，所以不推荐使用这种方式；第二种方式把命令用`$()`包围起来，区分更加命令，所以推荐使用这种方式。

例如，我在 code 目录中创建了一个名为 log.txt 的文本文件，用来记录我的日常工作。下面的代码中，使用 cat 命令将 log.txt 的内容读取出来，并赋值给一个变量，然后使用 echo 命令输出。

```shell
[root@izbp12mpi6ej8nhsdjjk4jz test]# mkdir code
[root@izbp12mpi6ej8nhsdjjk4jz test]# cd code/
[root@izbp12mpi6ej8nhsdjjk4jz code]# vim log.txt
[root@izbp12mpi6ej8nhsdjjk4jz code]# log=$(cat log.txt)
[root@izbp12mpi6ej8nhsdjjk4jz code]# echo $log 
XLP正在学习shell教程。
[root@izbp12mpi6ej8nhsdjjk4jz code]# log=`cat log.txt`
[root@izbp12mpi6ej8nhsdjjk4jz code]# echo ${log}
XLP正在学习shell教程。
```

### 只读变量

使用`readonly`命令可以将变量定义为只读变量，只读变量的值不能被改变。

下面的例子尝试更改只读变量，结果报错：

```shell
#!/bin/bash
myUrl="https://www.baidu.com"
readonly myUrl
myUrl="https://www.google.com"
```

运行脚背，结果如下：

```
./test.sh: line 4: myUrl: readonly variable
```

### 删除变量

使用`unset`命令可以删除变量。语法：

```shell
unset variable_name
```

变量被删除后不能再次使用；`unset`命令不能删除只读变量。

举个例子：

```shell
#!/bin/bash
myUrl="https://www.baidu.com"
unset myUrl
echo ${myUrl}
```

上面的脚本没有任何输出

### 变量类型

运行shell时，会同时存在三种变量：

1）局部变量

局部变量在脚本或者命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。

2）环境变量

所有的程序，包括shell启动的程序都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。

3）shell变量

shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行。

## Shell特殊变量

前面已经讲到，变量名只能包含数字，字母和下划线，因为某些包含其他字符的变量有特殊含义，这样的变量被称为特殊变量。

例如，$表示当前Shell进行的ID，即pid，看下面的代码：

```shell
[root@izbp12mpi6ej8nhsdjjk4kz ~]# echo $$
```

运行结果

```
3551
```

特殊变量列表

| 变量 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| $0   | 当前脚本的文件名。                                           |
| $n   | 传递给脚背或函数的参数。n是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是2。 |
| $#   | 传递给脚本或函数的参数个数。                                 |
| $*   | 传递给脚本或函数的所有参数。                                 |
| $@   | 传递给脚本或函数的所有参数。被双引号（`“ ”`）包括时，与`$*`稍有不同，下面将会将到。 |
| $?   | 上个命令的退出状态，或函数的返回值。                         |
| $$   | 当前shell进行ID。对于shell脚本，就是这些脚背所在的进程ID。   |

### 命令行参数

运行脚本时传递给脚本的参数称为命令行参数。命令行参数用`$n`表示，例如`$1`表示第一个参数，`$2`表示第二个参数，依次类推。

请看下面的脚本：

```shell
#!/bin/bash
echo "File Name: $0"
echo "First Paramter: $1"
echo "First Paramter: $2"
echo "Quoted Values: $@"
echo "Quoted Values: $*"
echo "Total Number of Paramter: $#"
```

运行结果：

```shell
[root@izbp12mpi6ej8nhsdjjk4jz test]# ./test.sh 张三 李四
File Name: ./test.sh
First Paramter: 张三
First Paramter: 李四
Quoted Values: 张三 李四
Quoted Values: 张三 李四
Total Number of Paramter: 2
```

### `$*`和`$@`的区别

`$*`和`$@`都表示传递给函数或脚本的所有参数，不被双引号（`“ ”`）包含时，都以"$1","\$2"..."\$n"的形式输出所有参数。

但是当它们被双引号（`“ ”`）包含时，“`$*`”会将所有的参数作为一个整体，以"$1 \$2"..."\$n"的形式输出所有参数；"`\$@`"会将各个参数分开，以“\$1” "\$2"..."\$n"的形式输出所有参数。

下面的例子可以清楚的看到`$*`和`$@`的区别：

```shell
#!/bin/bash
echo "\$*=" $*
echo "\"\$*\"=" "$*"

echo "\$@=" $@
echo "\"\$@\"=" "$@"

echo "print each param from \$*"
for var in $*
do
	echo "$var"
done

echo "print each param from \$@"
for var in $@
do
	echo "$var"
done

echo "print each param from \"\$*\""
for var in "$*"
do
	echo "$var"
done

echo "print each param from \"\$@\""
for var in "$@"
do
	echo "$var"
done
```

执行`./test.sh a b c d`，看到下面的结果

```shell
[root@izbp12mpi6ej8nhsdjjk4jz test]# ./test.sh a b c d
$*= a b c d
"$*"= a b c d
$@= a b c d
"$@"= a b c d
print each param from $*
a
b
c
d
print each param from $@
a
b
c
d
print each param from "$*"
a b c d
print each param from "$@"
a
b
c
d
```

### 退出状态

`$?`可以获取上一个命令的退出状态。所谓退出状态，就是上一个命令执行的返回结果。

退出状态是一个数字，一般情况下，大部分命令执行成功会返回0，失败返回1。

不过，也有一些命令返回其他值，表示不同类型的错误。

下面的例子中，命令成功执行：

```shell
[root@izbp12mpi6ej8nhsdjjk4jz test]# ./test.sh 张三 李四
File Name: ./test.sh
First Paramter: 张三
First Paramter: 李四
Quoted Values: 张三 李四
Quoted Values: 张三 李四
Total Number of Paramter: 2
[root@izbp12mpi6ej8nhsdjjk4jz test]# echo $?
0
```

失败的情况

```shell
[root@izbp12mpi6ej8nhsdjjk4jz test]# date "%y"
date: invalid date ‘%y’
[root@izbp12mpi6ej8nhsdjjk4jz test]# echo $?
1
```

所以`$?`可以用作判断条件，判断上一个命令执行成功的情况下才执行后续的操作。

## Shell替换

如果表达式中包含特殊字符，Shell将会进行替换，例如，在双引号中使用变量就是一种替换，转移字符也是一种替换。

举个例子：

```shell
#!/bin/bash
a=10
echo -e "Value of a is $a \n"
```

运行结果：

```shell
Value of a is 10
```

这里`-e`表示对转义字符进行替换，如果不适用`-e`选项，将会原样输出：

下面的转移字符都可以用在echo中：

| 转义字符 | 含义       |
| -------- | ---------- |
| `\\`     | 反斜杠     |
| \a       | 警报，响铃 |
| \b       |            |
| \f       |            |
| \n       |            |
| \r       |            |
| \t       |            |
| \v       |            |







## Shell运算符

Basg支持很多运算符，包括算数运算符，关系运算符，布尔运算符，字符串运算符和文件测试运算符。

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如awk和expr，expr最常用。

expr是一款表达式计算工具，使用它能完成表达式的求值操作。

例如，两个数相加：

```shell
#!/bin/bash
variable=`expr 2 + 2`
echo 'Total Value : '+$variable
```

运行脚本输出：

```
Total Value : 4
```

**两点注意：**

- 表达式和运算符之间要有空格，例如2+2是不对的，必须写成2 + 2，这与我们熟悉的大多数编程语言不一样。
- 完成的表达式要被``包含，注意这个字符串不是常有的单引号，在Esc键下边。

### 算数运算符

先来看一个使用算术运算符的例子

```shell
#!/bin/bash
a=10
b=20

var=`expr $a + $b`
echo a + b = $var

var=`expr $a - $b`
echo a - b = $var

var=`expr $a \* $b`
echo a \* b = $var

var=`expr $b / $a`
echo b / a = $var

var=`expr $b % $a`
echo b % a = $var

if [ $a == $b ]
then
    echo "a is equal to b"
fi

if [ $a != $b ]
then
    echo "a is not equal to b"
fi
```

运行结果

```
a + b = 30
a - b = -10
a * b = 200
b / a = 2
b % a = 0
a is not equal to b
```





## Shell for循环

与其他编程语言类似，Shell支持for循环。

for循环一般格式为：

```shell
for 变量 in 列表
do
    command1
    command2
    ...
    commandN
done
```

**列表是一组值（数字、字符串等）组成的序列，每个值通过空格分隔。**每循环一次，就将列表中的下一个值赋给变量。

in列表是可选的，如果不用它，for循环使用命令行的位置参数。

例如，顺序输出当前列表中的数字：

```shell
for loop in 1 2 3 4 5
do 
	echo "The value is : $loop"
done
```

运行结果：

```
The value is : 1
The value is : 2
The value is : 3
The value is : 4
The value is : 5
```

顺序输出字符串中的字符：

```shell
for str in “This is a string”
do
	echo $str
done
```

运行结果：

```
This is a string
```

显示主目录下以.bash开头的文件：

```shell
#!/bin/bash
for FILE in $HOME/.bash*
do
	echo $FILE
done
```

运行结果：

```
/root/.bash_history
/root/.bash_logout
/root/.bash_profile
/root/.bashrc
```



































