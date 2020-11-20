

## intptr_t、uintptr_t数据类型解析

intptr_t、uintptr_t数据类型是ISO C99定义的，具体代码在Linux平台的/usr/include/stdint.h头文件中。

该头文件中关于intptr_t、uintptr_t这两个数据类型的定义代码片段如下：

```c++
/* Types for `void *' pointers.  */
#if __WORDSIZE == 64
# ifndef __intptr_t_defined
typedef long int		intptr_t;
#  define __intptr_t_defined
# endif
typedef unsigned long int	uintptr_t;
#else
# ifndef __intptr_t_defined
typedef int			intptr_t;
#  define __intptr_t_defined
# endif
typedef unsigned int		uintptr_t;
#endif
```

在64位的机器上，intptr_t、uintptr_t分别为long int和unsigned long int的别名。

在32位的机器上，intptr_t、uintptr_t分别为int和unsigned int的别名。

目的：为了提高程序的可移植性（同样代码，在32位和64位机器上都可适应）。

## C++输出二进制数

```c++
#include <iostream>
#include <bitset>
using namespace std;
int main(){
	int x=178;
	//<?>中的参数是指定输出多少位 
	cout<<bitset<sizeof(x)*8>(x)<<endl;//int占4字节，一个字节8位，最终输出的是32个0或1
	return 0;
}
```

