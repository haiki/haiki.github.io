---
title: cmake
date: 2023-03-12 21:24:37
categories:
- CPP
tags:
- 编译
- 基础
---

想要根据项目定制 CmakeLists.txt，**首先需要理解 cmake 是什么，用来做什么？**

根据参考文档：[比较老的官方文档](https://cmake.org/cmake/help/cmake2.4docs.html)， [Mastering CMake](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Getting%20Started.html)，cmake practice：

**Cmake 是跨平台的 build system**。开发者在项目的每层 source tree 目录中包含平台无关的配置文件 CMakeList.txt，用来定义编译流程，并根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件。

那么，**makefile 是什么？用来做什么？**
makefile关系到了整个工程的编译规则。一个工程中的源文件不计数，其按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作(因为makefile就像一个Shell脚本一样，其中也可以执行操作系统的命令)

makefile带来的好处就是——“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率。

那么，**make 又是什么？**

> make是一个命令工具，是一个解释makefile中指令的命令工具，一般来说，大多数的IDE都有这个命令，比如：Delphi的make，Visual C++的nmake，Linux下GNU的make。
> 《引自 <https://blog.csdn.net/haoel/article/details/2886>》

> makefile文件由configure脚本运行生成，这就是为什么编译时configure必须首先运行的原因。《引自 <https://blog.csdn.net/qq_21950671/article/details/94456864>》

那么，**cmake 和 makefile 之间的关系是什么？**

由上文可知，CMake 是跨平台编译工具，需要编写 CMakeLists.txt 文件。用 cmake 命令可将 CMakeLists.txt 文件转化为 make 所需要的 Makefiles 文件。用 make 命令调用g++来编译源码，生成可执行程序或共享库。

那么，**编译又是什么？为什么要编译？**

程序中一切皆变量，CPU 访问内存时需要的是地址，而不是变量名和函数名！变量名和函数名只是地址的一种助记符，当源文件被编译和链接成可执行程序后，各种名称都会被替换成地址，而编译和链接过程的一项重要任务就是找到这些名称所对应的地址。

具体地，将源代码（.c/.cpp文件）编译成可执行文件，一般需要预处理、编译、汇编、链接四个步骤，每个步骤会分别调用预处理器、编译器、汇编器、链接器来完成。

![截屏2023-03-12 21.41.44.png](https://s2.loli.net/2023/03/12/b5X2RJIyHQBdNjc.png)

在Linux环境下，如果你安装了GCC编译器（gcc 与 g++ 分别是 gnu 的 c & c++ 编译器），你会看到在程序的安装目录下面会有各种二进制可执行文件：

- cpp：预处理器；
- ccl：编译器；
- as：汇编器；
- ld：链接器；
- ar：静态库制作工具

对应地，gcc/g++ 在执行编译工作的时候，总共需要4步：

- 预处理,生成 .i 的文件
- 将预处理后的文件转换成汇编语言, 生成文件 .s [编译器egcs？]
- 汇编代码经汇编器变为目标代码(机器代码)，生成 .o 的文件[汇编器as]
- 链接目标代码, 生成可执行程序 [链接器ld]

程序在编译过程中会分别使用这些工具，完成程序编译的每个流程。为了简化程序编译流程，GCC编译器一般会提供一个gcc命令。gcc 一次编译一个源代码文件，make 一次可编译多个源代码文件。

**最后，再回归cmake，cmake 怎么写，怎么用？**

- cmake 使用方法：
cmake 可执行文件是 CMake 命令行的接口，可在命令行中用 -D 选项配置项目；-i 选项允许用户以交互的方式配置。
`cmake [options] <path-to-source>`

`cmake [options] <path-to-existing-build>`

![截屏2023-03-12 21.46.17.png](https://s2.loli.net/2023/03/12/qbKAyuTLVz5rn6d.png)

——————————————————————————————————
reference：

- <https://www.hahack.com/codes/cmake/>

tutorial：makefile

- <https://www.zhaixue.cc/makefile/makefile-rule.html>

cmake

- <https://cmake.org/cmake/help/book/mastering-cmake/chapter/Getting%20Started.html>
