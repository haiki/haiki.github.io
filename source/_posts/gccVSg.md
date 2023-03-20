---
title: gccVSg++
date: 2022-02-14 19:44:47
categories:
- CPP
tags:
- 编译
- 基础
---

[TOC]

我们在编译c/c++代码的时候，有人用gcc，有人用g++，于是各种说法都来了，譬如c代码用gcc，而c++代码用g++，或者说编译用gcc，链接用g++，一时也不知哪个说法正确，如果再遇上个extern "C"，分歧就更多了，这里我想作个了结，毕竟知识的目的是令人更清醒，而不是更糊涂。

gcc and g++ are compiler-drivers of the **GNU Compiler Collection** (which was once upon a time just the GNU C Compiler).

GNU Compiler Collection： Referrers to all the different languages that are supported by the GNU compiler.

gcc: GNU C   Compiler

g++: GNU C++ Compiler

当使用编译工具时，自动根据文件类型，来决定调用哪个后台程序，此时会有点差别，（但是可以通过 `-x 语言` 这个参数来覆盖这个规定）

The probably most important difference in their defaults is which libraries they link against automatically.


According to GCC's online documentation link options and how g++ is invoked, g++ is equivalent to gcc -xc++ -lstdc++ -shared-libgcc (the 1st is a compiler option, the 2nd two are linker options). This can be checked by running both with the -v option (it displays the backend toolchain commands being run).



## 误区一:gcc只能编译c代码,g++只能编译c++代码

结论：两者都可以，但是请注意：

1. gcc 在编译 `*.c\*.cpp` 文件时，分别以 C 和 C++ 的方式编译。
2. g++ 也可以编译 `*.c\*.cpp` 文件，认为文件都是 cpp 的。
3. 如果使用 g++ 链接对象文件，会自动链接 std C++ 库，而 gcc 不会做这个，此时可以通过 `gcc -lstdc++` 参数来链接 c++ std 库。
4. gcc 编译 C 文件没有预定义的宏（has fewer predefined macros.）
5. gcc 编译 `*.cpp` and g++ 编译 `*.c\*.cpp` 文件时，有一些额外的宏( has a few xtra macros.)

Extra Macros when compiling *.cpp files:

```cpp
#define __GXX_WEAK__ 1
#define __cplusplus 1
#define __DEPRECATED 1
#define __GNUG__ 4
#define __EXCEPTIONS 1
#define __private_extern__ extern
```


6. 后缀为.c的，gcc把它当作是C程序，而g++当作是c++程序；
7. 后缀为.cpp的，两者都会认为是c++程序，注意，虽然c++是c的超集，但是两者对语法的要求是有区别的，例如下面的代码：
   - ~~如果按照C的语法规则，OK，没问题，但是，一旦把后缀改为cpp，立刻报三个错：这3个错分别对应前面红色标注的部分。可见C++的语法规则更加严谨一些。~~
   - 但是实操后发现，后缀为 .c 时，不论是gcc还是g++都是报错，但是错误的中间一句不一样，gcc 认为是隐式转换，g++ 认为是未声明
   - 
8. 编译阶段，g++会调用gcc，对于c++代码，两者是等价的，但是因为gcc命令不能自动和C++程序使用的库联接，所以通常用g++来完成链接，为了统一起见，干脆编译/链接统统用g++了，这就给人一种错觉，好像cpp程序只能用g++似的。

```c
#include <stdio.h>

int main(int argc, char* argv[]) {
   if(argv == 0) return;
   printString(argv);
   return;
}

int printString(char* string) {
  sprintf(string, "This is a test.\n");
}
```

gcc

![截屏2023-03-12 22.26.45.png](https://s2.loli.net/2023/03/12/7wpXANZvKlQOSxj.png)

g++

![截屏2023-03-12 22.27.08.png](https://s2.loli.net/2023/03/12/xQ4dHVyzakq3BvK.png)

cpp 后缀
![截屏2023-03-12 22.31.51.png](https://s2.loli.net/2023/03/12/GiLfmHuCh2wePK3.png)

## 误区二:gcc不会定义__cplusplus宏，而g++会

实际上，这个宏只是标志着编译器将会把代码按C还是C++语法来解释，如上所述，如果后缀为.c，并且采用gcc编译器，则该宏就是未定义的，否则，就是已定义。

## 误区三:编译只能用gcc，链接只能用g++

严格来说，这句话不算错误，但是它混淆了概念，应该这样说：编译可以用gcc/g++，而链接可以用g++或者gcc -lstdc++。

因为gcc命令不能自动和C++程序使用的库联接，所以通常使用g++来完成联接。但在编译阶段，g++会自动调用gcc，**g++ is equivalent to gcc -xc++ -lstdc++ -shared-libgcc** [参考](https://stackoverflow.com/questions/172587/what-is-the-difference-between-g-and-gcc)

## 误区四:extern "C"与gcc/g++有关系
实际上并无关系，无论是gcc还是g++，用extern "c"时，都是以C的命名方式来为symbol命名，否则，都以c++方式命名。试验如下：
me.h：
extern "C" void CppPrintf(void);

 

me.cpp:
#include <iostream>
#include "me.h"
using namespace std;
void CppPrintf(void)
{
     cout << "Hello\n";
}

 

test.cpp:
#include <stdlib.h>
#include <stdio.h>
#include "me.h"        
int main(void)
{
    CppPrintf();
    return 0;
}

 

1. 先给me.h加上extern "C"，看用gcc和g++命名有什么不同
[root@root G++]# g++ -S me.cpp     //g++的参数-S： 是指把文件编译成为汇编代码
[root@root G++]# less me.s
.globl _Z9CppPrintfv        //注意此函数的命名
        .type   CppPrintf, @function

[root@root GCC]# gcc -S me.cpp
[root@root GCC]# less me.s
.globl _Z9CppPrintfv        //注意此函数的命名
        .type   CppPrintf, @function
完全相同！
               
2. 去掉me.h中extern "C"，看用gcc和g++命名有什么不同
[root@root GCC]# gcc -S me.cpp
[root@root GCC]# less me.s
.globl _Z9CppPrintfv        //注意此函数的命名
        .type   _Z9CppPrintfv, @function

[root@root G++]# g++ -S me.cpp
[root@root G++]# less me.s
.globl _Z9CppPrintfv        //注意此函数的命名
        .type   _Z9CppPrintfv, @function
完全相同！

【结论】完全相同，可见extern "C"与采用gcc/g++并无关系，以上的试验还间接的印证了前面的说法：在编译阶段，g++是调用gcc的。

> 摘自 <https://blog.csdn.net/tjcwt2011/article/details/103410446>
> 部分参考 [stackoverflow](https://stackoverflow.com/questions/172587/what-is-the-difference-between-g-and-gcc) 上的回答
