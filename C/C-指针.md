# 指针



## 2、指针变量的基本概念

### 2.1、只用指针变量的例子

```c
#include <stdio.h>

int main() {
    int a = 100, b = 10;
    int *pointer_1, *pointer_2;
    pointer_1 = &a;
    pointer_2 = &b;
    printf("a=%d,b=%d\n", a, b);
    printf("*pointer_1=%d,*pointer_2=%d", *pointer_1, *pointer_2);
    return 0;
}
```

### 2.2、怎样定义指针变量

定义指针变量的一般形式：

```c
类型 *指针变量名;
```

例如：

```c
int *pointer_1,*pointer_2;
```

- int是位指针变量指定的“基类型”。
- 基类型指定指针变量可指向的变量类型。
- 如pointer_1可以指向整型变量，但不能指向浮点型变量。

### 2.3、怎样引用指针变量

指针相关运算符

1. `&` 取地址运算符

   &a是变量a的地址。

2. `*` 指针运算符（“间接访问”运算符）

   如果：p指向变量a，则*p就代表a。

   k = *p （把a的值赋给k）

   *p = 1; （把1赋给a）