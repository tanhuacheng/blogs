C99 的一些特性
====================================================================================================


## 复数

包含 `complex.h` 头

1. 宏 `complex`, `I` 被扩展为关键字 `_Complex`, `_Complex_I`

2. 支持复数的数学函数: 以 c 开头的 `ccos`, `csin`, `ccosh`, `cexp`, `cpow`, `creal`, `cimag` 等

gcc 仅支持 **complex type**, 不支持 **imaginary type**


## 指定初始化

``` c
int a[8] = {1, 2, 3, 4, 5, 6, 7, 8};
int b[8] = {[2] = 0x55, [5] = 0xaa};
int c[8] = {[0 ... 7] = 0xaa};

struct point {
    int x;
    int y;
};

struct point p1 = {.x = 1, .y = 3};
struct point p2 = {y: 3, x: 1};
```


## 变长数组(VLA)

1. 可以定义一个长度为变量的数组(这个数组的长度在运行时才确定), 也可以在结构体或联合体中使用 VLA

    ``` c
    void foo(int n)
    {
        int x[n]; // 不能初始化, 例如: int x[n] = {0}; 是错误的

        ...
    }

    void bar(int n)
    {
        struct {
            int x[n];
        } s;

        ...
    }
    ```

2. 可以作为函数的参数

    ``` c
    void foo(int n, char a[n]); // foo(n, a);
    void bar(int n; char a[n], int n); // bar(a, n);
    ```


## 柔性数组(flexible array member)

在结构体成员的最后存放一个零长度的数组(标准规定不能写零, 即 `[]`, 但 gcc 允许 `[0]`)

``` c
struct line {
    int length;
    char data[]; // 如果使用 gcc 编译器, 则可以写成 char data[0];
};
```

柔性数组不占用实际的内存空间, 如上定义的结构体中 `data` 就是结构体对象后面紧接着的内存地址

``` c
int length = 10;
struct line *line = calloc(1, sizeof(struct line) + length);
line->length = length;
line->data[length - 1] = expr
```


## 增加了一些标准头

`stdint.h` `stdbool.h` `tgmath.h` `complex.h` `inttypes.h` 等


## 支持 64 位整型

`long long int` 或 `unsigned long long int`, 整数后面加 `LL` 或 `ULL`. 该类型至少占用 64 位, 例如:
``` c
long long int x = 0LL;
```


## 复合常量(Compound Literals)

允许定义一个匿名的结构体或数组

``` c
foo((int[]){1, 2, 3});
select(0, NULL, NULL, NULL, &(struct timeval){.tv_sec = 2});
```

但是在 gcc 中使用 --std c99(gnu99, c11, gnu11) 时不能使用以下方式(使用 -std gnu90(默认) 时可以):

``` c
int a[] = (int[]){1, 2, 3};
```


## 定义变量

1. 不要求局部变量一定在第一条语句之前

    ``` c
    {
        ...
        int i;
        ...
    }
    ```

2. 可以在 for 循环中定义循环变量了

    ``` c
    for (int i = 0; i < N; i++) {
        ...
    }
    ```


## 可变参宏

``` c
#define LOG(fmt, args...) printf(fmt, ##args)
```
或
``` c
#define LOG(fmt, ...) printf(fmt, ##__VA_ARGS__)
```


## #pragma once

指示编译器不用重复引入一个头文件, 传统的做法是:

``` c
#ifndef PROJECT_HEADNAME
#define PROJECT_HEADNAME
...
#endif
```

现在只需要在头文件开始的地方加入一行代码即可:

``` c
#pragma once
```


## 整型

表达式求值

    优先级:
        1) 没有两个有符号数整型拥有相同的优先级, 即使它们有相同的表示
        2) 如果一个有符号整型比另一个有符号整型更大, 更大的具有更高的优先级
        3) long long > long > int > short > signed char > _Bool
        4) 一个无符号整型和与它对应的有符号整型具有相同的优先级
        5) char 具有与 signed char 和 unsigned char 相同的优先级
        6) enum 和用于表示它的整型具有相同的优先级
        7) 如果一个标准整型与一个扩展整型具有相同的大小, 则标准类型具有更高的优先级
        8) 如果两个扩展类型具有相同的大小, 它们之间的优先级由实现定义, 但仍然遵从其它整数转换优先级
           决定的规则

    公共类型:
        1) 如果两个操作数都有相同的类型, 公共类型就是这个类型
        2) 如果一个操作数是整型而另一个操作数是浮点类型, 公共类型是浮点类型
        3) 如果两个操作数都是有符号整型或无符号整型, 公共类型是更高优先级的类型
        4) 当一个操作数是有符号整型而另一个是无符号整型时, 规则如下:
            1)) 如果无符号整型的优先级高于或等于有符号类型, 公共类型是无符号类型
            2)) 否则, 如果有符号整型可以表示无符号整型的值时, 公共类型是有符号类型
            3)) 否则, 公共类型是对应的有符号类型的无符号类型

        INT32_C(-1) < UINT32_C(1) 结果是 false (公共类型是 uint32_t)

标准头 `stdint.h`

    定义了各种大小整型的 typedef 和扩展成这些类型最大值的宏(对于有符号类型, 还有最小值), 以及一些
    其它的实用宏

        int_leastN_t, uint_leastN_t
        int_fastN_t, uint_fastN_t
        intN_t, uintN_t

    其中 N 表示该整型的大小的十进制整数, 通常有 8, 16, 32, 64; least 表示最小的由 N 指定的大小的类型;
    fast 表示能表示由 N 指定的大小的运算最快的类型. 由于有的实现不支持 intN_t, uintN_t, 所以这些都是
    可选的

    特殊的整型

        intptr_t, uintptr_t: 它们的大小足够存储一个指向某个对象的指针(也许不足以存储一个函数指针)
        intmax_t, uintmax_t: 实现支持的最大整型

没有在 `stdint.h` 中定义的整型

    ptrdiff_t
    size_t
    wchar_t

各类型值的限制

    INT_LEASTN_MAX INT_LEASTN_MIN UINT_LEASTN_MAX
    INT_FASTN_MAX INT_FASTN_MIN UINT_FASTN_MAX
    INTN_MAX INTN_MIN UINTN_MAX

    INTPTR_MAX INTPTR_MIN UINTPTR_MAX
    INTMAX_MAX INTMAX_MIN UINTMAX_MAX

    PTRDIFF_MAX PTRDIFF_MIN
    SIZE_MAX
    WCHAR_MAX WCHAR_MIN

构建类型常量(宏)

    INTN_C() UINTN_C()
    INTMAX_C() UINTMAX_C()

标准头 `inttypes.h`

    允许 printf 和 scanf 使用 stdint.h 中定义的整型

        PRIsN
        PRIsLEASTN
        PRIsFASTN
        PRIsMAX
        PRIsPTR

    其中 s 是由 printf 的整数格式说明符, 即 d, i, o, u, x 或 X, 如果使用 scanf 则把 PRI 换为 SCN 即
    可(除 s 不能是 X 外)

        printf("%" PRIdMAX, (intmax_t)x);
        scanf("%" SCNd8, &((int8_t)x));
