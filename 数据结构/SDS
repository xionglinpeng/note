# SDS

[TOC]

##　1、Introduction

sds全称为*简单动态字符串*（Simple dynamic strings），是Redis中为了表示字符串对象而定义的一种数据结构。

> IMPORTANT: 当前对SDS的分析是基于Redis 6.2.6，不同的Redis版本实现上有些差异，但大体是一样的。

## 2、Why need sds?

Redis是基于**C语言**实现的一个分布式缓存NoSQL数据库，因此SDS自然也是基础C语言实现的，SDS的存在主要解决了以下几个问题。

1. 常数时间获取字符串长度

   在C语言中，对字符串的定义是通过字符数组（`char[]`）定义的，以空字符（`\0`）标识结尾。如果要获取一个字符串的长度，需要遍历整个字符数组才能得到，其时间复杂度是O(n)。而sds定义了`len`成员，可以通过该成员直接获取字符串的长度，其时间复杂度是O(1)。而在Redis中获取字符串长度的操作是相当普遍的，因此采用sds可以有效提升效率。

2. 杜绝缓冲区溢出

   在C语言中，常见的字符串操作函数如下所示：

   ```c
   char* strcpy(char* dest, const char* src);
   char* strcat(char* dest, const char* src);
   ```

   这些函数在执行时，若`dest`没有足够的空间来容纳`src`，将会导致缓冲区溢出，从而意外修改其他数据。而使用sds进行拼接赋值等操作时，sds会首先检查是否有足够的空间来进行操作，如果不足，sds会自动进行扩容以满足操作的需要。因此在编程中使用sds即不需要手动进行扩容，也不会发生缓冲区溢出的错误。

3. 减少内存重分配次数

   在C语言中，字符串实际上是使用**null**字符`\0`截止的一维字符数组，所以当每次需要增加或缩短字符串时，都需要重新分配或释放内存空间。如果字符串发生N次变化，则需要进行N次内存分配/释放操作。而在sds中通过*空间预分配*和*惰性空间释放*两种策略来减少内存重分配次数。

   - **空间预分配**：在sds进行空间扩容时，不仅会为修改分配所需要的空间，还会分配额外未使用的空闲空间。

     *规则如下：*

     ★ 修改后的字符串大小小于1M时，则空闲空间直接翻倍。

     ★ 修改后的字符串大小大于或等于1M时，每次扩容空闲空间增加1M。

     sds定义了`alloc`成员，用以记录整个sds可用于字符串分配空间（已使用+未使用）的大小，每次进行字符串长度增加时，会先看剩余空闲空间是否足够使用，如果足够，则直接使用，反之，则扩容新的空闲空间。

   - **惰性空间释放**：惰性空间的释放是用于优化SDS字符串的缩短操作：当SDS的API需要缩短保存的字符串时，程序并不会立即使用内存重分配来回收缩短后多出来的字节，而是使这些多出来的空间被设置为空闲空间，并重置`len`属性，而空闲空间将等待将来使用。

     > **惰性分配**
     >
     > 惰性分配是指在实际需要的时候才分配资源，例如各种常见的懒加载机制。严格来说，任何时候资源分配得越晚越好。通过将资源分配延迟到实际需要时，可以减少启动时间，甚至在从未实际使用该资源的情况下完全消除分配。相反，也可以预先分配稍后所需要的资源，以牺牲启动时间为代价，使稍后的执行效率更高，并且还可以避免稍后程序在执行过程中分配失败的可能性。
     >
     > **惰性空间**
     >
     > SDS采用了预先分配资源的策略，而这里的资源就是指在扩容时的冗余缓冲区或者说未被使用的缓冲区，又称惰性空间。

4. 二进制安全

   C字符串只支持ASCII字符，中间不能存在空字符。而SDS可以存储二进制数据，通过`len`来判断是否到达末尾。

5. 支持部分C函数

   SDS中的buf之所以以空字符结尾，就是为了支持部分c函数，如下所示：

   ```cpp
   //通过c语言API直接对比buf和c字符串
   strcasecmp(sds->buf, "hello world");
   ```

## 3、总体布局

​																						图A![](images/sds-1.png)

sds主要分为两大部分：`sdshdr`和`buf[]`。其中`sdshdr`是sds的头部（header），`buf[]`是真正存储字符串的body。

header中主要包含以下几部分：

- `len`：记录字符串的真正长度，不包含结尾的空字符（`\0`）。通过该字段可以快速的获取字符串的长度，时间复杂度：T(n)=O(1)。

- `alloc`：记录缓冲区的大小，不包含结尾的空字符（`\0`）。`alloc-len`即为空闲的缓冲区。

- `flags`：表示header的类型，是一个`char`类型，占1个字节，其中3 bits用于标识类型，5 bits未被使用。可以认为是指定了当前sds字符串的长度范围。取值范围0~4。

  > *flags的取值（源码位于[sds.h](https://github.com/redis/redis/blob/unstable/src/sds.h)）：*
  >
  > ```c
  > #define SDS_TYPE_5  0
  > #define SDS_TYPE_8  1
  > #define SDS_TYPE_16 2
  > #define SDS_TYPE_32 3
  > #define SDS_TYPE_64 4
  > ```

sds一共定义了5种不同类型的header，其目的是为了满足不同长度的字符串使用不同的header，从而节省内存。

- `sdshdr5`：从未被使用过，只是直接访问标记字节。
- `sdshdr8`：表示长度小于或等于256字节的字符串值。
- `sdshdr16`：表示长度小于或等于64KB的字符串值。
- `sdshdr32`：表示长度小于或等于4GB的字符串值。
- `sdshdr64`：表示长度小于或等于16EB的字符串值（16EB是理论值，实际场景基本不可能，由于硬件的限制，在Linux和Windows等系统上最大只能存储256GB或者128GB等大小的内存）。

`buf[]`是柔性数组，不会占用header的内存空间，它只是标识结构体后面存储的是一个数组。

> 在sds存储字符串时，字符串并不会真正存储到`buf[]`中，而是按内存分配的方式存储紧接header之后，所以，这里`buf[]`仅仅是占位符的意思，即获取字符串的时候并不会通过`buf[]`获取，而是通过获取到header之后首地址指针，并结合`len`的属性的方式获取。

**Header的获取**

因为header类型有5种，因此当需要获取sds的header时，就需要知道header的类型——即header的`flags`字段。如上图A所示的sds结构，通过指针`s`向前移动一个字节`s[-1]`就可以获取当前sds的header的`flags`值。而要得到指针`s`，只需要通过指针*sh+header的长度*即可。

## 4、柔性数组

在C语言标准C99中，结构体中的最后一个成员允许是大小未知的数组，这个成员被称为**柔性数组**。

柔性数组包含以下几个特点：

1. 结构体中的柔性数组成员的前面必须至少有一个其他成员。
2. `sizeof`函数获取结构体大小时不包含柔性数组的内存（柔性数组在结构体中不占内存）。
3. 包含柔性数组成员的结构体用`malloc()`函数进行内存动态分配。

`sdshdr#T`中的`buf[]`就是柔性数组。

如下代码所示，定义了结构体`struct1`和`struct2`：

```c
struct struct1 {        |         struct struct2 {
    int a;              |         	 int a;
    int b[];			|			 int *b;
}                       |         }
```

通过`sizeof`函数获取`struct1`的大小为4，获取`struct2`的大小为8。

## 5、源码分析

相关源码位于[sds.h](https://github.com/redis/redis/blob/unstable/src/sds.h)，[sds.c](https://github.com/redis/redis/blob/unstable/src/sds.c)，[sdsalloc.h](https://github.com/redis/redis/blob/unstable/src/sdsalloc.h)。

主要关注以下四点，分别是*创建新的sds*，*缓冲区扩容*，*惰性空间释放*和*回收空闲空间*。

下面是sds中定义的header结构体源码：

```c
/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

###  5.1、创建新的sds

一个新的sds的创建所对应的函数如下所示：

```c
sds sdsnewlen(const void *init, size_t initlen) {
    return _sdsnewlen(init, initlen, 0);
}

sds sdstrynewlen(const void *init, size_t initlen) {
    return _sdsnewlen(init, initlen, 1);
}
```

它们主要调用了`_sdsnewlen`函数。`_sdsnewlen`函数源码如下：

```c
const char *SDS_NOINIT = "SDS_NOINIT";

/* 使用'init'指针和'initlen'参数所指定的内容创建一个新的sds字符串。如果init指针指向NULL，
 * 则字符串初始化为零字节。如果'init'为SDS_NOINIT，则不初始化缓冲区。
 *
 * 字符串总是以null结尾（\0，所有的sds字符串都是如此），即使你创建了一个字符串：
 *     mystring = sdsnewlen("abc",3);
 *
 * 可以使用printf()函数打印字符串，因为字符串末尾有一个隐式的\0。但是字符串是二进制安全的，
 * 中间可以包含\0字符串，因为sds字符串的长度存储在header的len成员中。*/
sds _sdsnewlen(const void *init, size_t initlen, int trymalloc) {
    void *sh;
    sds s;
    //根据字符串的长度获取对应的Header类型
    char type = sdsReqType(initlen);

    /* 当initlen等于0时，表示创建空字符串，其目的常常是为了执行追加操作。
     * 因此强制使用类型8，因为类型5不能处理这个。*/
    if (type == SDS_TYPE_5 && initlen == 0) type = SDS_TYPE_8;
    //获取对应Header类型的字节大小
    int hdrlen = sdsHdrSize(type);
    unsigned char *fp; /* flags pointer. */
    size_t usable; /* 等价于alloc字段，整个SDS可用的大小 */

    assert(initlen + hdrlen + 1 > initlen); /* Catch size_t overflow */
    // hdrlen+initlen+1 → 头长度+初始长度+\0 = SDS长度
    //分配内存，返回sh指针，sh指向header的起始位置。并且将整个sds的长度赋值给&usable
    sh = trymalloc?
        s_trymalloc_usable(hdrlen+initlen+1, &usable) :
        s_malloc_usable(hdrlen+initlen+1, &usable);
    //内存分配失败，直接返回NULL
    if (sh == NULL) return NULL;
    //init指针指向SDS_NOINIT，则将init设置为NULL，即不初始化缓冲区
    if (init==SDS_NOINIT)
        init = NULL;
    else if (!init)
        //将这个SDS空间先全部清空
        memset(sh, 0, hdrlen+initlen+1);
    s = (char*)sh+hdrlen; // sh的位置+头的长度 = buf的起始指针位置
    fp = ((unsigned char*)s)-1; // s的位置向前一个字节就是flag的位置
    usable = usable-hdrlen-1; // 此时usable为buff的长度了
    if (usable > sdsTypeMaxSize(type))
        usable = sdsTypeMaxSize(type); // buff不能超过flags的限制
    switch(type) {
        case SDS_TYPE_5: {
            *fp = type | (initlen << SDS_TYPE_BITS);
            break;
        }
        case SDS_TYPE_8: {
            SDS_HDR_VAR(8,s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
        case SDS_TYPE_16: {
            SDS_HDR_VAR(16,s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
        case SDS_TYPE_32: {
            SDS_HDR_VAR(32,s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
        case SDS_TYPE_64: {
            SDS_HDR_VAR(64,s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
    }
    if (initlen && init)
        memcpy(s, init, initlen); //将字符串拷贝到s的位置
    s[initlen] = '\0'; //末尾设置为空
    return s;
}
```

`_sdsnewlen`函数包含三个参数：

1. `*init` - 执行初始化字符串的指针。

2. `initlen` - 初始化字符串的长度。

3. `trymalloc` - 类型为`int`，但在`_sdsnewlen`函数中用作`bool`判断。

   当`trymalloc`等于`1`时尝试分配内存，如果分配失败，返回NULL；

   当`trymalloc`等于`0`时尝试分配内存，如果分配失败，抛出异常。

### 5.2、缓冲区扩容

sdscatlen函数位于[sds.c](https://github.com/redis/redis/blob/unstable/src/sds.c)文件中，它的作用是将指定的字符串追加到某个sds的后面，简而言之，就是字符串追加。

sdscatlen函数源码如下：

```c
/* 将len字节的t（追加的字符串指针）指向的指定二进制安全字符串追加到指定的sds字符串's'的末尾。
 *
 * 调用之后，传递的sds字符串不再有效，所有引用必须用调用返回的新指针替换。 */
sds sdscatlen(sds s, const void *t, size_t len) {
    //当前sds的长度
    size_t curlen = sdslen(s);
    //保证有足够的内存空间用于追加字符串（t）
    s = sdsMakeRoomFor(s,len);
    //如果sds扩展失败，则直接返回空
    if (s == NULL) return NULL;
    //追加t到s的末尾，s+curlen就是s的末尾指针
    memcpy(s+curlen, t, len);
    //设置新的sds的header的len属性
    sdssetlen(s, curlen+len);
    //末尾设置为\0
    s[curlen+len] = '\0';
    return s;
}
```

`sdscatlen`函数包含3个参数：

- `s`：原sds的指针。
- `*t`：追加字符串的指针。
- `len`：追加字符串的长度。

流程很简单，不在过多描述，主要的关键点在于`sdsMakeRoomFor`函数，这个函数的作用是保证有足够的内存空间用于追加字符串，源码如下：

```c
/* 扩大sds字符串末尾的空闲空间，这对于避免重复追加到sds时的重复重新分配非常有用。*/sds sdsMakeRoomFor(sds s, size_t addlen) {    return _sdsMakeRoomFor(s, addlen, 1);}/* 与sdsMakeRoomFor()不同，该函数只会扩容刚好需要的大小。 */sds sdsMakeRoomForNonGreedy(sds s, size_t addlen) {    return _sdsMakeRoomFor(s, addlen, 0);}
```

可以看到，它们仅仅只是转调了`_sdsMakeRoomFor`函数。而它才是正在执行sds扩容操作的函数。

`_sdsMakeRoomFor`函数源码如下：

```c
/* 扩大sds字符串末尾的空间，以便调用者确定调用此函数之后字符串末尾至少有addlen字节的空间可以使用， * 并且还要加上一个nul字节。如果已经有足够的空闲空间，该函数将不做任何操作直接返回，如果没有足够的 * 空闲空间，则将分配缺少的空间，甚至更多: * 当greedy等于1时，增加一定的冗余空间，以避免将来需要对增量增长进行重新分配。 * 当greedy等于0时，增加刚好addlen的空间，以便给addlen使用。 * * Note: 该函数不会改变sdslen()所计算得到sds字符串的*length*，只改变所拥有的空闲缓冲区的空间。*/sds _sdsMakeRoomFor(sds s, size_t addlen, int greedy) {    void *sh, *newsh;    //可用的空闲缓冲区空间    size_t avail = sdsavail(s);    size_t len, newlen, reqlen;    //新的header类型和当前sds的header类型（oldtype）    char type, oldtype = s[-1] & SDS_TYPE_MASK;     int hdrlen;    size_t usable;	//如果空闲缓冲区大于等于追加的字符串的长度，则什么都不做，直接返回当前sds    if (avail >= addlen) return s;    //原sds的长度    len = sdslen(s);    //原sds的头指针    sh = (char*)s-sdsHdrSize(oldtype);    //原sds的字符串长度+新追加的字符串的长度 = 新sds字符串的长度    reqlen = newlen = (len+addlen);    assert(newlen > len);   /* Catch size_t overflow */    //greedy等于1时增加冗余空间    if (greedy == 1) {        //如果新sds字符串的长度小于1M，则空闲缓冲区增加一倍        if (newlen < SDS_MAX_PREALLOC)            newlen *= 2;        else            //如果大于1M，则直接增加1M空闲缓冲区            newlen += SDS_MAX_PREALLOC;    }	//根据新的sds的字符串长度计算出其header类型    type = sdsReqType(newlen);    /* 不使用SDS_TYPE_5，因为SDS_TYPE_5不能记住缓冲区空间信息，即没有len和alloc成员*/    if (type == SDS_TYPE_5) type = SDS_TYPE_8;    //新的header类型的大小    hdrlen = sdsHdrSize(type);    assert(hdrlen + newlen + 1 > reqlen);  /* Catch size_t overflow */    //如果新的header类型与旧的header类型一致    if (oldtype==type) {        //新的header类型与旧的header类型一致，因此header不变（内存地址），重新分配缓冲区空间。        newsh = s_realloc_usable(sh, hdrlen+newlen+1, &usable);        //内存分配失败直接返回空        if (newsh == NULL) return NULL;        //重新设置s指针的引用        s = (char*)newsh+hdrlen;    } else {        //由于header大小改变，需要将字符串向前移动，不能使用realloc，即分配新的内存空间        newsh = s_malloc_usable(hdrlen+newlen+1, &usable);        //内存分配失败直接返回空        if (newsh == NULL) return NULL;        //将旧的sds的字符串拷贝到新的sds的buf[]中。注意加上了末尾的\0        memcpy((char*)newsh+hdrlen, s, len+1);        s_free(sh);        //重新设置s指针的引用        s = (char*)newsh+hdrlen;        //设置flags        s[-1] = type;        //设置len        sdssetlen(s, len);    }    //计算alloc    usable = usable-hdrlen-1;    //超出header最大可表示空间，则usable直接设置为最大可用空间    if (usable > sdsTypeMaxSize(type))        usable = sdsTypeMaxSize(type);    //设置alloc    sdssetalloc(s, usable);    return s;}
```

`_sdsMakeRoomFor`函数包含三个参数：

- `s`：原sds的指针。
- `addlen`：追加字符串的长度。
- `greedy`：是否追加冗余空间。1→是；0→否。

### 5.3、惰性空间释放

`sdsclear`函数修改sds字符串，使其为空(零长度)，但是现有的缓冲区都不会被丢弃，而是被设置为空闲空间，以便下一个追加操作可以直接分配。

```c
void sdsclear(sds s) {    //直接将len设置为0    sdssetlen(s, 0);    //缓冲区首字符设置为空    s[0] = '\0';}
```

### 5.4、回收空闲空间

```c
/* 重新分配sds字符串，使其在末尾没有空闲空间。 所包含的字符串将保持不变，但下一个连接操作将需要重新分配。 * * 调用之后，传递的sds字符串不再有效，所有引用必须用调用返回的新指针替换。 */sds sdsRemoveFreeSpace(sds s) {    void *sh, *newsh;    char type, oldtype = s[-1] & SDS_TYPE_MASK; //旧SDS header类型    int hdrlen, oldhdrlen = sdsHdrSize(oldtype); //旧SDS header长度    size_t len = sdslen(s);    //可用空间的大小    size_t avail = sdsavail(s);    //原SDS头部起始指针(sh)    sh = (char*)s-oldhdrlen;    //如果没有可用空间，则直接返回    if (avail == 0) return s;    //新SDS Header类型和长度    type = sdsReqType(len);    hdrlen = sdsHdrSize(type);    /* 如果类型相同，或者至少仍然需要足够大的类型，则只需要realloc()，让分配器只在真正需要时进行复制。     * 否则，如果变化很大，我们手动重新分配字符串以使用不同的头文件类型。*/    if (oldtype==type || type > SDS_TYPE_8) {        //重新以sh指针开始申请空间，即复用原来的空间，因为这里是字符串缩短，原来的空间肯定是足够用。        /* 注意：执行realloc()的条件是flags类型没变，或者类型大于SDS_TYPE_8，也就是说，当类型不         * 管变没变，只要大于SDS_TYPE_8，就会重新申请（缩短）空间，就可能导致flags与实际的字符串大小不         * 匹配。例如剩下的字符串大小对应的flags是SDS_TYPE_16，但flags却是SDS_TYPE_32。这里是基于延         * 迟加载的考虑：让分配器只在真正需要时进行重新设置，例如追加字符串时调用函数_sdsMakeRoomFor()         * 扩容时。*/        newsh = s_realloc(sh, oldhdrlen+len+1);        //内存分配失败直接返回空        if (newsh == NULL) return NULL;        //新SDS的s指针        s = (char*)newsh+oldhdrlen;    } else {        //重新分配一块内存空间为新的SDS        newsh = s_malloc(hdrlen+len+1);        //内存分配失败直接返回空        if (newsh == NULL) return NULL;        //将旧SDS的字符串拷贝到新SDS的缓冲区中        memcpy((char*)newsh+hdrlen, s, len+1);        //释放旧SDS字符串        s_free(sh);        //新SDS的s指针        s = (char*)newsh+hdrlen;        //设置新SDS→flags        s[-1] = type;        //设置新SDS→len        sdssetlen(s, len);    }    //设置SDS→alloc    sdssetalloc(s, len);    return s;}
```
