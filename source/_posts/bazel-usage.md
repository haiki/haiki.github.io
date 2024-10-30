---
title: bazel_usage
date: 2022-06-03 09:31:09
categories:
- 工具
tags:
- bazel
---

官网入口：~~https://docs.bazel.build/versions/main/bazel-overview.html~~ https://bazel.build/start



# 初识 Bazel 



## Bazel 是什么
开源的**编译和测试**的工具（类似Make，Maven，Gradle）。

- **High-level build language**
  - 使用易读，抽象的编译语言，以高层次的语义描述项目构建属性。
  - 在 libraries，binaries，scripts，date sets 的概念上运行，用户无需单独写规则调用编译器/链接器。
- **Bazel is fast and reliable**
  - Bazel 缓存之前编译的结果，追踪修改的文件内容和构建命令，从而知道哪些内容需要重新构建，并只构建修改的内容和命令。想要加速构建，**可设置高并行和增量的方式**。

- **Bazel is multi-platform**
  - 可在多种平台（macos，windows，linux）运行。
  - 支持为同个项目在多种平台（桌面，服务端，移动端）构建二进制文件和可部署的包。
- **Bazel scales.**
  - 支持多达100k+源文件的编译，支持多个代码库，可供多人（万+）使用。
- **Bazel is extensible**
  - 支持[多种语言](https://bazel.build/rules)（C++，Go，Java等），且可以做延伸去支持其他语言或框架。



## 如何用 Bazel

- 下载 Bazel
  - 推荐用 Bazelisk 在 Ubuntu，Windows，MacOs 安装 Bazel，官方团队维护。
    - MacOs：brew install bazelisk。
    - [bazelisk](https://github.com/bazelbuild/bazelisk) 是用go写的bazel的wrapper，会自动选择当前工作目录适合的 Bazel 版本，从官方服务器下载，将所有的命令行参数透传给真正的Bazel二进制文件。所以可以像调用 bazel 一样调用 bazelisk。
- 新建项目的 [WORKSPACE](https://bazel.build/concepts/build-ref#workspaces) 
  - 工作目录，Bazel 寻找构建的输入和 BUILD 文件，并存储构建输出。

- 写 BUILD 文件
  - 定义了需要构建以及如何构建的内容，可以使用[Starlark](https://bazel.build/rules/language) 声明构建 target，[example](https://github.com/bazelbuild/bazel/blob/master/examples/cpp/BUILD)。
  - 构建 target 声明了构建需要的输入和依赖，构建它们需要的 rule，以及配置 rule 的 options。
  - 构建 rule 声明了 Bazel 用来构建的工具，如编译器，链接器以及其配置，bazel 配备了多种构建 rule，可覆盖所支持平台上，所支持语言的大部分类型。
- 通过[命令行](https://bazel.build/reference/command-line-reference)运行 Bazel 
  - build 命令 https://docs.bazel.build/versions/main/guide.html#specifying-targets-to-build



 **除了编译，Bazel还可以执行[测试](https://bazel.build/reference/test-encyclopedia)，并[查询](https://bazel.build/docs/query-how-to) 构建链路，追踪代码中的依赖。**



## Bazel 构建流程

当执行构建/测试，Bazel 如下执行：

- **加载**与目标相关的 BUILD 文件。
- **分析**输入，以及输入的[依赖](https://bazel.build/concepts/dependencies)，应用特定的构建规则，生成[行为图](https://bazel.build/rules/concepts#evaluation-model) 。
  - 行为图表示构建artifacts，artifacts 之间的关系，以及 Bazel 要执行的构建行为。
  - 根据行为图，Bazel 可以[追踪](https://bazel.build/docs/build#build-consistency)文件内容以及构建/测试行为的改变，曾经的构建结果，也可以[追踪代码的依赖](https://bazel.build/docs/query-how-to)。
- 在输入上**执行**构建行为，直到输出最后的构建产物。



Bazel 会将之前的构建工作缓存，只构建/测试修改了的内容。为了确保正确性，可以通过 sandBoxing [Hermetically](https://bazel.build/concepts/hermeticity#overview) 构建和测试，minimizing skew and maximizing [reproducibility](https://bazel.build/docs/build#correct-incremental-rebuilds) 。



# Bazel 基础
Bazel 构建 workspace 中以目录树组织的源文件。workspace 中的源文件由嵌套层级的 packages 组织，每个 packag 是一个目录，包含相关的源文件和一个BUILD 文件。BUILD 文件定义了可以从源文件编译出的输出。



## [Workspaces,Packages, targets](https://bazel.build/concepts/build-ref)

### Workspace

- Bazel 的构建基于工作区(workspace) 概念。每个使用 bazel 的项目，在根目录下都必须有一个 WORKSPACE 文件，WORKSPACE 文件所在的目录就是项目的根目录。
  - 如果**当前 WORKSPACE 文件**所在的**子目录中包含 WORKSPACE 文件**，Bazel 构建时会忽略子目录，子目录成为另一个 workspace 。
  - WORKSPACE  文件的别名是 WORKSPACE.bazel，两者都存在时，使用 WORKSPACE.bazel。
- WROKSPACE 文件可以是空的，也可以包含对[外部的依赖](https://bazel.build/docs/external)。



#### Repository

代码组织在 repositories 中。

- 包含 WORKSPACE 文件的目录是 main repository **的根，称作 “@”。**

- 其他（外部） repositories 由 WORKSPACE 文件中的 workspace rules 定义。
  - 外部 repositories 通常包含自己的 WORKSPACE 文件，而 Bazel 会忽略这些文件。
  - In particular, repositories depended upon transitively are not added automatically.
- [Repository Rules](https://bazel.build/rules/lib/repo )



### Packages

Repository 中主要的代码组织单元是 package。package 包含相关文件的集合，以及这些文件如何产出输出的说明。

- 包含 BUILD/BUILD.bazel 文件的目录是一个 package。

- 一个 package 中包含这个目录/子目录下的所有文件，subpackage 除外，这样就没有文件/目录会成为两个不同 package 的成员 。

  - 包含 BUILD 文件的子目录称作 subpackage。

  

[package groups](https://bazel.build/reference/be/functions#package_group) 是 packages 的集合，其目的是限制特定 rules 的可访问性。

- package groups 由 `package_group ` 函数定义，包含三个属性：
  1. 此 group 中包含的 packages 列表；
  2. packages 的名字；
  3. 其包含的其他 package groups。
- 引用 package group 的方式是：
  1. 通过 rules 的 `visibility` 属性；
  2. `package` 函数的 `default_visibility` 属性 。
- **不生成或者消费文件。**



### Targets

Package 是 targets 的容器，targets 定义在 package 的 BUILD 文件中。大部分 targets 是 files 或者 rules。

- files 又可以分成两类：源文件（source files，包含在 repo 中），生成的文件（Generated files，由源文件通过特定的 rules 生成，不包含在 repo 中）。~~targets 中定义了包括：files，rules 和 package groups。~~

- Rule ：定义了一系列输入和输出之间的关系，包含了从输入得到输出的必要步骤。
  - 输入可以是 source file/generated file ，类型不重要，重要的是文件的内容。因此将复杂的源文件替换为 generated file 很方便，反之亦然。也可以避免一个文件修改导致所有编译链接关系的改变 。
  - 一个的输入可能是另一个的输出，从而形成关系链，而输出只是 generated files。
  - 一个 rule 的输入可能包含其他 rules：直觉上来说，比如 C++的 library rule A 以 library rule B 作为输入，那么编译期间，B 的头文件对 A 可用，链接期间，B 的符号对 A 可用，运行期间，B 的运行时数据对 A 可用。
  - rules 的不变性：一条 rule 生成的文件，只属于包含这条 rule 的 package。尽管一条 rule 的输入可能会来自其他 package，当前 rule 无法为其他 package 生成文件。



## Labels 
所有的 targets 只会属于一个 package。target 的名字叫作 label。每个 label 唯一标识 target。 

- 典型的 label：  `@myrepo//my/app/main:app_binary `
  - `@myrepo//`：**repository 名**。
  - 当 label 被所在的 repository 中使用，repo 的名字可以使用缩写 `//`。因此，在 `@myrepo` 中，label 常写为 `//my/app/main:app_binary` 。
- `my/app/main`： 从 repository 的根到 package 的相对路径，是 un-qualified 的 **package 名**。
  - repository 名 +  un-qualified package 名 组成 **fully-qualified 的 package 名**： `@myrepo//my/app/main `
  -  当 label 被所在 package 中引用， package 名（以及冒号）可以被省略。上述 label 可以写为 ：
    - `app_binary` 
    - `:app_binary`
  - 不成文规定。有冒号的用在 rule 中，*无冒号的用在 file 中*，但也无所谓啦。
- `app_binary `： un-qualified target 名
  - 当和 target 名和 package 路径的最后一个目录一致时，可以省略，即 label 表示为：
    1. `//my/app/lib`
    2.  `//my/app/lib:lib`
- file target 的名字：对于 package 子目录中的 file target，其名称为相对于所在 package root（包含 BUILD 文件的目录） 的路径。
  - file target 的名字为：`//my/app/main:testdata/input.txt` ，文件位于 repo 的子目录 `my/app/main/testdata`  下。



注意：

- `//my/app` 不是 package 名。
  - labels 总是以 repo 的标识（//）开头，但是 package 名不是。
  - `my/app` 才是包含 `//my/app/lib` (a.k.a.` //my/app/lib:lib`）的 package。
  - 错误说法：`//my/app` 指向一个 package ，或者是指向 package 中的所有 target。
  - 正确说法：`//my/app`  等价于 `//my/app:app` ，即命名了在当前 repo 中，`my/app` package 下的 app target。

- **相对的 labels 不能用来指向其他 package 中的 targets，这种情况下必须指定 repo 和 package。**
  - 比如有 package `my/app` 和 package `my/app/testdata`，后者包含了文件` testdepot.zip`，那么在` //my/app:BUILD` 中引用 后者时：
    - 正确做法是：`//my/app/testdata:testdepot.zip` ；
    - 而不是 `testdata/testdepot.zip`  # Wrong: testdata is a different package。
    - 如果是同一个 package，应该可以用相对 label 吧？
  
- *以 `@//` 开头的 labels 是引用 main repo，从外部 repo 引用时也可以用这种形式。*
  - `@//a/b/c`  和 `//a/b/c` 从外部 repo 引用时是不同的：
    - 前者引用 main repo；
    - 后者由外部 repo 自己寻找 //a/b/c。
  -  当在 main repo 中写 rule 引用 main repo 的 target，且将会被外部 repo 引用时，就有关系了。?

更多引用 target 的方法见[详情](https://bazel.build/docs/build#specifying-build-targets)。



### label 的命名规范

#### Target names —  package-name:target-name

- `target-name` 是 package 内 target 的名字。可包含大小写和数字，以及标点符号： `!%-@^_"#$&'()*-+,;<=>?[]{|}~/.`。
- rule 的名字是 `BUILD` 文件中 rule 声明时的 `name` 属性。
-  file 的名字是相对于 package 的路径，必须是正常的形式：
  - 不可以以 `/` 开头或者结尾，错误示例：`/foo`，`foo/`。
  - 不可用包含` //`，错误示例：`//foo`。
  - 不可以有相对路径引用：错误示例：`..` 或者 `./`。
    - 不可用 `..` 引用其他 package 的 files，可以用 `//package-name:filename`。
-  file target 中 可以使用` /`，但是在 rule 的名字中不要使用`/`：
  - 尽管没有 package `//foo/bar/wiz`，`//foo/bar/wiz` 也表示 `//foo/bar/wiz:wiz`。
  - 尽管  `bar/wiz` target 存在，`//foo/bar/wiz` 也无法表示为 `//foo:bar/wiz`。
  - 但是对于一些场合必须，也是允许的。比如特定 rules 的名字，必须和源文件名一致，而这些文件存在于 package 的子目录中。



#### Package names — //package-name:target-name

- package 的名字是包含 BUILD 文件的目录，相对于 main repository 的目录名。如 `my/app`。
- package 名可包含大小写和数字，以及标点符号 `/, -, ., @,_`。不可以以 `/` 开始或者结束，也不要包含 `//`。
- Bazel 支持在 workspace 的根 package 下声明 target，如`//:foo`，但是尽量让这个 package 不为空，这样每个 package 都有可描述的名字。



### Rule

- rules 声明了输入和输出间的关系，以及构造输出的步骤。

- rules 有很多类型（称作 *rule class*），可产生**编译好的的二进制可执行文件，库，测试用的可执行文件**以及[其他支持的输出](https://bazel.build/reference/be/overview)。
- BUILD 文件通过调用 rules 声明 targets。

rule 的名字（即 name 属性）必须是合法的 target 名字。

- 每条 rule 的调用都有 name 属性（其值必须是合法的 targe name），声明了package 中的一个 target。
  - 有时候，rule 的名字比较随意，and more interesting are the names of the files generated by the rule, and this is true of genrules. For more information, see [General Rules: genrule](https://bazel.build/reference/be/general#genrule).
  - 有时候，rule 的名字比较重要，比如 `*_binary` and `*_test` rules，rule 的名字决定了构建输出的可执行文件的名字。
- 每条 rule 都有多个属性，从[构建百科全书](https://bazel.build/reference/be/overview)可以看到一些 rule 和对应的属性。
  - 每个属性有name 和 type，可看作属性是从键（名称）到可选类型值的字典。
  - type 的可选类型包括 integer，label，list of label，string，list of string，output label，list of output label。
  - `src` 属性的 type 是  "list of labels"，每个 label 是当前 rule 的输入。

```python
cc_binary(
    name = "my_app",
    srcs = ["my_app.cc"],
    deps = [
        "//absl/base",
        "//absl/strings",
    ],
)
```



target 组成的有向无环图称作  *target graph* or *build dependency graph*，是 [Bazel Query tool](https://bazel.build/docs/query-how-to) 执行的领域。



## BUILD文件

build文件就是真正定义编译规则的文件了，每个目录下都有一个，每个源文件都要在BUILD中定义它的编译规则。



`BUILD` files are evaluated using an imperative language, [Starlark](https://github.com/bazelbuild/starlark/)。文件中的 rules 被解释为顺序的 statements。

- 一般来说， 变量的声明要在使用之前，顺序很重要，但是 BUILD 文件中只包含构建 rules 的声明，所以顺序不是很重要。只需要保证用到的 rules 被声明过了。
- 为了鼓励代码和数据的分离，BUILD 文件中： 不允许有函数的定义，`for` statements，或者 `if` statements（但是 list comprehensions and `if` expressions  是可以的），不支持 `*args` and `**kwargs` 参数，参数必须明确支持。
- 可以在 `.bzl` 文件中定义函数。 


Starlark 中的程序不可以执行任意的 I/O，因此输入都是定的，使得构建都是可再现的，即 [Hermeticity](https://bazel.build/concepts/hermeticity)。



### Loading an extension

Bazel extensions 是以 `.bzl` 结尾的文件，可使用 `load` statement 从 extension 导入符号。

`.bzl` 文件中以`_` 开头的符号不会被导出，也不会被其他文件加载。文件可见性不会影响加载：因此不需要使用 `exports_files` 让 `.bzl` 文件可见。？？

```python
load("//foo/bar:file.bzl", "some_library")
```

- 从 `//foo/bar:file.bzl` 加载代码，并把 `some_library` 符号加入环境。
- 此方式可用来加载 新的 rules，函数，或常量（string，list等）
- 可以在 `load` 中新增其他的参数，导入更多的符号。参数必须是字符串常量，不可以是变量。
- `load` statement 必须在顶层，不可以在函数体内。
- `load` 的第一个参数是标识 `.bzl` 的 label，如果是相对的 label，则解析为包含当前 `.bzl` 文件所在的 package（不是目录） ，load statements 中的相对 label 需要有前导 `:` 。



```python
load("//foo/bar:file.bzl", library_alias = "some_library")
```

- load 支持别名，因此可以给导入的符号赋不同的名字。



```python
load(":my_rules.bzl", "some_rule", nice_alias = "some_other_rule")
```

- 可以在一个 load statement 中定义多个别名。





### 构建规则的类型

The majority of build rules come in families, grouped together by language. 



- `*_binary` rules构建给定语言的可执行程序。如 label `//my:program` 的结果存放在    `$(BINDIR)/my/program`。
  - 有的语言，会将 rule 的 data 属性下的文件（以及依赖rule的这种文件），放到 runfiles 目录，便于部署到生产。
- `*_test` rules 是 `*_binary` rule 的特化，用于自动化测试。
  - 和 二进制文件一样，tests 也有 runfiles 树，其中只包含测试运行时必要的文件。比如 `cc_test(name='x', data=['//foo:bar'])` may open and read `$TEST_SRCDIR/workspace/foo/bar` during execution. （每种语言都有自己的函数去获得`$TEST_SRCDIR` 的值，但是都等同于直接使用环境变量）
- *_library rules 声明了每个模块的构建规则，可以依赖其他 libraries，binaries 和 tests 可以依赖 libraries。



*在 cc_binary 中定义可执行程序的编译规则，在cc_library中定义库的编译规则（即.o文件），在trpc_proto_library中定义pb文件的编译规则。*



## 依赖

如果 A 在构建或者执行时，需要 B，那么 target A 依赖 target B。这种 依赖关系构成 [DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph)，称作 **依赖图**。

直接依赖：在依赖图中一跳可到达。

可传递依赖：依赖图中多跳到达。

构建时，存在两种依赖图：实际（ actual ）依赖，声明（declared）依赖。



### 实际依赖和声明依赖

target X 实际依赖 target Y，当且仅当为了 X  编译成功， Y 必须存在，且是最新的。**在代码中实际用了的。**

“built” 代表：generated, processed,compiled, linked, archived, compressed, executed等在构建过程中经常发生的任务。

target X 声明依赖 target Y，当且仅当 X 所在的  package 中，有一条  X 到 Y 的依赖边。**在rule 规则中写了的。**



*为了正确编译，实际依赖的图 A 必须是声明依赖图 D 的子图 。也就是说，A 中直接相连的节点 x--> y. 在 D 中也必须直接相连，称作  D 是 A 的 overapproximation。overapproximation不要太过度，否则冗余的声明依赖会使得编译很慢，二进制文件很大。*

> For correct builds, the graph of actual dependencies *A* must be a subgraph of the graph of declared dependencies *D*. That is, every pair of directly-connected nodes `x --> y` in *A* must also be directly connected in *D*. It can be said that *D* is an *overapproximation* of *A*.
>
> **Important:** *D* should not be too much of an overapproximation of *A* because redundant declared dependencies can make builds slower and binaries larger.



**写 BUILD 文件时，每条 rule 都必须显示声明所有实际的直接依赖。不要试图列出所有的非直接依赖。**

这是一条很重要的规则，否则构建可能会依赖之前的一些操作，或者会依赖到一些传递声明的 target，导致未知错误。虽然 bazel 会做检查并报告错误，但是有时候并不全面。



传递依赖带来的常见问题：一个文件中的代码可能会使用非直接依赖中提供的代码，非直接依赖不会写在 BUILD 文件中，因此无法追踪这些文件的修改，比如文件的依赖，a 声明依赖b，b 声明依赖 c，a 中实际依赖 b，实际依赖 c，那么当 b 中没有实际依赖 c 时，会影响到 a 的 构建。



### 依赖的类型

大部分构建 rule 在声明不同的依赖时，有都[通用的属性](https://bazel.build/reference/be/common-definitions)，以及 [rule 特定的依赖](https://bazel.build/reference/be)，例如 `compiler` / `resources`。

- 通用属性
  - `src`
  - `deps`
  - `data`：有的构建 target 运行时需要数据文件，数据文件不是源文件，所以不会影响 target 如何被构建。比如 单元测试需要比对函数的输出和文件的内容，构建单元测试 target 时不需要数据文件，但是运行时需要。The same applies to tools that are launched during execution.
    - 构建系统在单独的目录运行测试，只有列举在 data 中的文件时可用的。



These files are available using the relative path `path/to/data/file`. In tests, you can refer to these files by joining the paths of the test's source directory and the workspace-relative path, for example, `${TEST_SRCDIR}/workspace/path/to/data/file`.



### 使用 labels 引用目录

- `data` labels 引用目录，最好不要以/. 或者 / 结尾的方式。不推荐示例：
  -  `data = ["//data/regression:unittest/."]`
  - `data = ["testdata/."]`
  - `data = ["testdata/"]`

这种直接写目录的方式看起来很方便，直接能用目录下的所有文件，但是不要这样用：

- 为了确保正确的增量构建，以及测试的重新执行，构建系统必须要知道所有的输入文件。
- 对于直接包含整个目录的方式，编译系统只检查目录有没有变化（是否有增删文件），不会探测对单个文件的内部修改。
- 因此，不要直接把目录作为编译系统的输入，应该将目录内的文件列举出来，明确说明。或者用 `glob()` 函数，如 `data = glob(["testdata/**"])`，用 `** `强制 glob 函数递归。



当一些场景必须使用目录 labels（即以目录结尾），而其名称不符合 label 语法，那么遍历文件，或者使用 glob 函数会导致非法 label 错误，则必须使用目录 label。此时，对于父 package，不可以用相对的 `../` 路径，应该使用绝对路径如`//data/regression:unittest/`



注意：**directory label** 只对 data 依赖有效。



当外部 rule，比如 test ，需要使用多个文件时，需要显示声明对所有文件的依赖，可以在 BUILD 文件中用 `filegroup` 将文件打包在一起。即可在test 中引用 label `my_data`。

```python
filegroup(
        name = 'my_data',
        srcs = glob(['my_unittest_data/*'])
)
```





~~Bazel 中 funciton 的BUILD 百科
Rules
推荐的 rules ：https://docs.bazel.build/versions/main/rules.html#recommended-rules
Bazel binary 中配备的 native rule 不需要用 load 声明，native rules 对于 BUILD 文件是全局可用的，可在 .bzl 文件的 native 模块找到。~~



## Visibility

package and subpackages 的信息，详见[概念和术语](https://bazel.build/concepts/build-ref)。

Visibility 控制了当前的 target 是否可以被其他 package 中的 target 依赖。从而区分 library API 以及实现细节。关闭可见性检查： `--check_visibility=false`。



### Visibility specifications

rule target 的 `visibility` 属性中包含多个 labels，表示此 target 对这些 labes 可见。同一个 package 中的 target 是互相可见的。

labels 的形式：

- `"//visibility:public"`：任何人都可以使用此 target。
- `"//visibility:private"`：只有此 package 可以使用此 target。
- `"//foo/bar:__pkg__"`：`//foo/bar`（不包含其 subpackage） 下定义的 targets 可以使用此 target。`__pkg__` 是 package 中所有 targets 的特殊句法。
- `"//foo/bar:__subpackages__"`：`//foo/bar`下定义的（包括直接或间接 package 的）所有 targets，可以使用此 target。
- `"//foo/bar:my_package_group"`：[package group](https://bazel.build/reference/be/functions#package_group) 中的所有 package 可访问。



example

```python
//some/package

cc_library (
  name = "mytarget",
  src = ["xx",],
  visibiliry = [
    ":__subpackages__",
    "//tests:__pkg__"
  ]
)

# mytarget 可以被 //some/package/... 下的所有 target 以及 //tests/BUILD 中定义的 target 使用，但是不能被 //tests/integration/BUILD 中的 target 使用。
```



`package_group`  targets 本身没有 visibility 属性，总是 publicly visible。

Visibility cannot be set to specific non-package_group targets. That triggers a "Label does not refer to a package group" or "Cycle in dependency graph" error.



### Visibility of a rule target/generated file target



rule target 中没有设置 visibility 属性，则可见性由BUILD 文件中的 package statement 设定，key 是  [`default_visibility`](https://bazel.build/reference/be/functions#package.default_visibility) 。 没有声明 default_visibility 时，默认是 `//visibility:private`。



config_setting 可见性默认不强制生效。

`--incompatible_enforce_config_setting_visibility` and `--incompatible_config_setting_private_default_visibility` provide migration logic for converging with other rules.



- If `--incompatible_enforce_config_setting_visibility=false`, every `config_setting` is unconditionally visible to all targets.

- Else if `--incompatible_config_setting_private_default_visibility=false`, any `config_setting` that doesn't explicitly set visibility is `//visibility:public` (ignoring package [`default_visibility`](https://bazel.build/reference/be/functions#package.default_visibility)).
- Else if `--incompatible_config_setting_private_default_visibility=true`, `config_setting` uses the same visibility logic as all other rules.

Best practice is to treat all like other rules: explicitly set `visibility` on any `config_setting` used anywhere outside its package.

最佳实践：

- 将  `config_setting` targets 和其他 rules 一样对待：为每个用到 `config_setting` 的package，显示设置  `visibility` 。



### Visibility of a source file target

默认情况下，源文件 targets 只在同一个 package 中可见，如果其他 package 也想访问此源文件，使用 [`exports_files`](https://bazel.build/reference/be/functions#exports_files)。使用此方法时，如果设置了 visibility 属性，则应用；否则文件就是 public 的， `default_visibility` 设置被忽略。

如果可以，推荐将文件包裹成 library 或者其他类型的 rule，而不是直接用源文件。

example：

File `//frobber/data/BUILD`：

```python
exports_files(["readme.txt"])
```

File `//frobber/bin/BUILD`：

```python
cc_binary(
  name = "my-program",
  data = ["//frobber/data::readme.txt"],
)
```





If the flag [`--incompatible_no_implicit_file_export`](https://github.com/bazelbuild/bazel/issues/10225) is not set, a legacy behavior applies instead.

With the legacy behavior, files used by at least one rule target in the package are implicitly exported using the `default_visibility` specification. See the [design proposal](https://github.com/bazelbuild/proposals/blob/master/designs/2019-10-24-file-visibility.md#example-and-description-of-the-problem) for more details.



### Visibility of bzl files

load statement 暂时不应用 visibility，因此可以在 workspace 的任意地方加载 bzl 文件。



However, users may choose to run the Buildifier linter. The [bzl-visibility](https://github.com/bazelbuild/buildtools/blob/master/WARNINGS.md#bzl-visibility) check provides a warning if users `load` from beneath a subdirectory named `internal` or `private`.



### Visibility of implicit dependencies 

有的 rules 有隐式依赖。如 c++ 的 rule 可能隐式依赖 c++ 编译器。当前的隐式依赖和一般依赖的处理方式一致，需要对 rule 的所有实例可见，通过 using [`--incompatible_visibility_private_attributes_at_definition`](https://github.com/bazelbuild/proposals/blob/master/designs/2019-10-15-tool-visibility.md). 修改此行为。



### 最佳实践

- 避免将 default visibility 设置为 public
- 使用  `package_group` 在多个 targets 中共享 vidibility 声明。当多个 BUILD 中的 targets 需要暴露给同一个 package 的集合时，此方法很有用。
- 当 deprecating 一个 target 时，使用更细粒度的可见性说明，只将 visibillity 限制给当前的用户，避免新的依赖。



## Platforms





## Hermeticity







## cc_libraryxxxx

头文件的包含检查。=》只适用于直接包含。
● 在编译中用到的头文件，必须在cc_* rule 的 hdrs 或者 srcs 中被声明。=》强制
● 对于 cc_library rules，hdrs 中的头文件，组成了这个 library 的公共接口
	○ 此 library 的 hdrs 和 srcs 中的文件可直接包含 hrds 中的头文件
	○ 其他 library 的 deps 如果依赖此 library，其 srcs 和 hdrs 也能包含此 library 的hdrs中的头文件
● srcs 中的头文件只能被此 library 的 hdrs 和 srcs 中的文件包含。
● 将文件放在 srcs 或者 hdrs，需要考虑，你是否希望此 library 的用户可以直接包含这个头文件，类似编程语言中的 public 和 private。
cc_binary 和 cc_test rule 没有对外的接口，因此没有 hdrs 属性，直接放在 srcs 中就行。
对.cc文件的编译，可以递归地包含在递归的 deps 必报中的所有 cc_library 中的 hdrs 和 srcs 中的文件。





# Output Directory Layout xxx

https://bazel.build/docs/output_directories
条件
当前布局
● Bazel 必须在包含 WORKSPACE 的目录 or 其子目录中被调用
● 输出根目录（outputRoot）默认为：
	○ linux：~/.cache/bazel
	○ macos：/private/var/tmp
	○ Windows：defaults to %HOME%, %USERPROFILE%
	○ 如果设置了环境变量 $TEST_TMPDIR ，对 bazel 做测试，则覆盖默认
● 用户构建状态存在于outputUserRoot：outputRoot/_bazel_$USER，其中包含：
	○ install/md5(Bazel installation manifest)：installBase
	○ 如果 bazel 在 workspace 目录（或者符号链接）/home/user/src/my-project，outputBase 是目录名的 MD5 hash。
		■ 可以使用 --output_base 覆盖原有的 output base 目录
		■ 可以使用 --output_user_root 启动选项覆盖默认的 install base 和 output base目录
● bazel-<workspace-name>，bazel-out，bazel-testlogs，bazel-bin 在 workspace 目录，都是一些符号链接，指向 output 目录中 target-specific 的目录。这些文件只为用户方便使用，Bazel 不使用。 

布局图

bazel clean
● bazel clean 在 outputPath 和 action_cache 目录上执行 ，并移除 workspace 符号链接。--expunge 清空整个 outputBase。





#### [Wrokspace Rules](https://bazel.build/reference/be/workspace)。xxxx

当你使用bazel build xxxx命令进行编译时，bazel 会以 WORKSPACE 文件所在目录作为根目录（寻找输入和BUILD文件）进行编译，并存储编译结果。
WORKAPCE 文件里定义了bazel项目的一些基本信息，和项目需要的外部依赖（比如当前项目依赖外部项目中的目标，或者从网上下载项目）
● 主项目的BUILD文件可以使用 WORKSPACE 中的名字，依赖外部 target
● WORKSPACE 文件中的语法和BUILD类似，但是还允许其他的规则
● repository rules（workspace rules）
	○ 内置规则：https://docs.bazel.build/versions/main/be/workspace.html
		■ local_repository
		■ new_local_repository
	○ starlark 中内嵌的 repository 规则（git or http）：https://docs.bazel.build/versions/main/repo/index.html
	○ 用户自定义 repository 规则：https://docs.bazel.build/versions/main/skylark/repository_rules.html
外部依赖定义方式
详细见：https://docs.bazel.build/versions/main/external.html
建议 http_archive > git_repository > new_git_repository
依赖其他 Bazel 项目
可以使用 local_repository，git_repository, http_archive，分别代表：从本地文件系统软连接，引用 git 仓库，从网上下载。
● WORKSPACE 中写法：
local_repository(
    name = "coworkers_project",
    path = "/path/to/coworkers-project", 
)

● BUILD 中写法：@coworkers_project//foo:bar 。外部项目的命名必须是合法的workspace名字，即用 xx_xx，而不是 xx-xx。
依赖其他非 Bazel 项目
new_local_repository，new_git_repository， http_archive
依赖外部包
比如maven项目





## 认识.bazelrc文件 xxxx

bazel在编译时可以指定编译选项，包括gcc的选项以及 bazel 自身的选项。否则，每次使用bazel build xxxx命令进行编译时，你都需要指定编译选项，比如c++17。







# 编写 BUILD 文件



## BUILD 文件的风格

使用 [Buildifier](https://github.com/bazelbuild/buildifier) 格式化 BUILD 文件。

- 文件的结构：

  - 描述/注释
    - 单独的注释下面用空行隔开
    - 某个元素的注释紧挨着

  - load

  - package

  - 对 rules/macros 的调用

- 引用当前package的 targets：

  - 使用相对于package 的路径，不要用`..` 
  - 生成的文件，引用时以 `:` 开头，标识其不是源文件

  - 源文件不要以 `:` 开头

  - rules 需要以`:`  开头

    ```python
    cc_library(
        name = "lib",
        srcs = ["x.cc"],
        hdrs = [":gen_header"],
    )
    
    genrule(
        name = "gen_header",
        srcs = [],
        outs = ["x.h"],
        cmd = "echo 'int x();' > $@",
    )
    ```

- target 命名
  - 命名需要有描述性
    - 如果 target 包含源文件，target name 需要和 源文件名一致。
    - 与 package 同名的 target 在最后一层目录，其名字需要具有描述性，如果描述和target不一致，不要用同名 target。
    - 当使用 eponymous target 时，建议用 短名字：(`//x` instead of `//x:x`)。
    - 如果在同一个 package 中，建议用局部引用： (`:x` instead of `//x`)。
    - 不要使用保留的关键字。如 all，`__pkg__`等。
  - google 的推荐命名做法

- visibility
  - 尽量使用准确的可见性，只有对外暴露的才使用 public。
- dependencies
  - 尽量限制为直接依赖。代码中有的依赖中也写。
  - 本 package 的依赖优先写，不要用绝对 package 名。
  - 将通用的依赖放在一个变量中。
- Globs
  - 使用 `[]` 标识没有 targets，不要使用 glob 去匹配 nothing。
  - 不要用递归的 globs 匹配源文件，如 `glob(["**/*.java"])`)。
  - 建议每个文件夹下都放一个 BUILD 文件，生成依赖图。
  - 非递归的 globs 是可接受的。
- 其他惯例
  - 使用大写和下划线声明常量（如  `GLOBAL_CONSTANT`），使用小写和下划线声明变量（如  `my_variable`）。
  - labels 不要切分，便于替换等操作。
  - name 属性的名字需要是 常量字符串。
  - 设置布尔值属性时，使用 true/false 而不是 0/1。



## 共享变量

- 当一条内容会被多次使用，可以在当前 BUILD 文件中定义变量（全局常量一般用大写字母），如 `COPTS = ["-DVERSION=5"]` 。

- 在多个 BUILD 文件间共享变量，需要放在 .bzl 文件中。

  - .bzl 文件中的定义（变量和函数）可以用在 BUILD 文件中。

  In `path/to/variables.bzl`, write:

  ```python
  COPTS = ["-DVERSION=5"]
  ```

  Then, you can update your `BUILD` files to access the variable:

  ```python
  load("//path/to:variables.bzl", "COPTS")
  
  cc_library(
    name = "foo",
    copts = COPTS,
    srcs = ["foo.cc"],
  )
  ```





## 外部依赖



bazel 可以依赖其他项目的 targets，其他项目的依赖叫做外部依赖（*external dependencies*）。

`WORKSPACE/WORKSPACE.bazel` 文件描述了 bazel 如何获取其他项目的源文件，称作 *repository rules* /*workspace rules*

- 其他项目也包含了多个 BUILD 文件描述自己的 targets。
- mian 项目中的 BUILD 文件可以使用 WORKSPACE 中定义的名字，来引用外部的 targets。



Bazel comes with a few [built-in repository rules](https://bazel.build/reference/be/workspace) and a set of [embedded Starlark repository rules](https://bazel.build/rules/lib/repo). Users can also write [custom repository rules](https://bazel.build/rules/repository_rules) to get more complex behavior.



### bazel 支持的外部依赖类型

1. Depending on other Bazel projects
   - 可以用 [`local_repository`](https://bazel.build/reference/be/workspace#local_repository), [`git_repository`](https://bazel.build/rules/lib/repo/git#git_repository) or [`http_archive`](https://bazel.build/rules/lib/repo/http#http_archive) ，从本地文件系统软链，或者引用 git repository，或者下载。
2. Depending on non-Bazel projects
   - 为此项目的依赖写 BUILD 文件。build_file 
3. Depending on external packages
   - Maven artifacts and repositories



- 获取依赖：

  - 一般通过 `bazel build`

  - 要为一些 targets 预拉取，使用 `bazel fetch`

  - 无条件拉取所有依赖，使用 `bazel sync`

  - As fetched repositories are [stored in the output base](https://bazel.build/docs/external#layout), fetching happens per workspace

- Shadowing dependencies
  - 尽可能在自己的项目中只依赖一种版本
  - xxx
- 通过命令行覆写 repositories
  - 设置 [`--override_repository`](https://bazel.build/reference/command-line-reference#flag--override_repository) flag. 使用此flag，只改变外部 repository 的内容，而不改变源码。
  - to override `@foo` to the local directory `/path/to/local/foo`, pass the `--override_repository=foo=/path/to/local/foo` flag。
  - 用途
    - debugging，不去拉远端，而改本地，让调试更方便。
    - Vendoring，没法发起网络调用的环境下，使用本地 repository。
- Transitive dependencies
  - Bazel only reads dependencies listed in your `WORKSPACE` file. If your project (`A`) depends on another project (`B`) which lists a dependency on a third project (`C`) in its `WORKSPACE` file, you'll have to add both `B` and `C` to your project's `WORKSPACE` file. This requirement can balloon the `WORKSPACE` file size, but limits the chances of having one library include `C` at version 1.0 and another include `C` at 2.0.
- 缓存外部依赖
  - Bazel 默认只会对修改的部分重新下载。如果强制重新下载，使用  `bazel sync`。
- 布局
  - 外部依赖下载到 external 文件夹。
  - `bazel clean` 只清除了软链，而 `bazel clean --expunge` 清除了所有外部 artifacts。

- 线下构建
  - xxx

### 最佳实践

- A repository rule should generally be responsible for:

  - Detecting system settings and writing them to files.

  - Finding resources elsewhere on the system.

  - Downloading resources from URLs.

  - Generating or symlinking BUILD files into the external repository directory.

- 尽量避免使用 `repository_ctx.execute` 

  - 当使用非bazel 化的 c++ library，优先推荐使用 `repository_ctx.download()` ，之后写 BUILD 文件构建它，而不是运行   `ctx.execute(["make"])`。

- Prefer [`http_archive`](https://bazel.build/rules/lib/repo/http#http_archive) to `git_repository` and `new_git_repository`：

  - Git repository 依赖于系统中的 git，而 HTTP 下载器是集成在 bazel 中的，没有系统依赖。
  - `http_archive` 支持 `urls` 列表作为 mirrors, 而  `git_repository` 只支持单个的 `remote`。





## 通过 Bzlmod 管理依赖 new xxx







# 运行 bazel 



## 用 bazel 构建 

### 可用的 bazel 命令

- [`analyze-profile`](https://bazel.build/docs/user-manual#analyze-profile): Analyzes build profile data.
- [`aquery`](https://bazel.build/docs/user-manual#aquery): Executes a query on the [post-analysis](https://bazel.build/docs/build#analysis) action graph.
- [`build`](https://bazel.build/docs/build#bazel-build): 构建特定目标
- [`canonicalize-flags`](https://bazel.build/docs/user-manual#canonicalize-flags): Canonicalize Bazel flags.
- [`clean`](https://bazel.build/docs/user-manual#clean): 移除输出文件，并停止 server
- [`cquery`](https://bazel.build/docs/cquery): Executes a [post-analysis](https://bazel.build/docs/build#analysis) dependency graph query.
- [`dump`](https://bazel.build/docs/user-manual#dump): Dumps the internal state of the Bazel server process.
- [`help`](https://bazel.build/docs/user-manual#help): 打印命令的 help 信息，或者索引
- [`info`](https://bazel.build/docs/user-manual#info): 打印 bazel server 的运行时信息
- [`fetch`](https://bazel.build/docs/build#fetching-external-dependencies): 拉取 target 的所有外部依赖
- [`mobile-install`](https://bazel.build/docs/user-manual#mobile-install): 在 mobile 设备上安装apps
- [`query`](https://bazel.build/docs/query-how-to): 执行依赖图的查询
- [`run`](https://bazel.build/docs/user-manual#running-executables): 运行特定 target
- [`shutdown`](https://bazel.build/docs/user-manual#shutdown): 停止 bazel server
- [`test`](https://bazel.build/docs/user-manual#running-tests): 构建和运行特定的 test target
- [`version`](https://bazel.build/docs/user-manual#version): 打印 bazel 的 version 信息



### 获取帮助

- `bazel help command`: 打印 command 的 help 和 options
- `bazel help`[`startup_options`](https://bazel.build/docs/user-manual#startup-options): Options for the JVM hosting Bazel.
- `bazel help`[`target-syntax`](https://bazel.build/docs/build#specifying-build-targets): 解释特定 target 的句法
- `bazel help info-keys`: info 命令中使用的 keys



command：bazel tool 执行的许多函数。`bazel build` 和 `bazel test` 用的比较多。

### 构建单个 target

用 label 标识要构建的 target。

- loads
- analyzes
- execute



### 构建多个 target

bazel 允许多种方法，去声明要构建的 target，叫做 target patterns（Target patterns are a generalization of the label syntax for *sets* of targets, using wildcards. 单个targe也是一种通配 target 的特例），用于 `build`, `test`,   `query` commands。

- foo/... 表示 所有 package 的通配符
- :all 表示某个 package 中所有的 targets
- :* 是 :all 的超集，包含了 file 和 rule

| `//foo/bar:wiz`         | Just the single target `//foo/bar:wiz`.                      |
| ----------------------- | ------------------------------------------------------------ |
| `//foo/bar`             | Equivalent to `//foo/bar:bar`.                               |
| `//foo/bar:all`         | **All rule targets** in the package `foo/bar`.               |
| `//foo/...`             | **All rule targets** in **all packages** beneath the directory `foo`. |
| `//foo/...:all`         | **All rule targets** in **all packages** beneath the directory `foo`. |
| `//foo/...:*`           | **All targets (rules and files)** in **all packages** beneath the directory `foo`. |
| `//foo/...:all-targets` | **All targets (rules and files)** in **all packages** beneath the directory `foo`. |
| `//...`                 | **All targets** in packages **in the workspace**. This does not include targets from [external repositories](https://bazel.build/docs/external). |
| `//:all`                | **All targets** in the **top-level package**, if there is a `BUILD` file at the root of the workspace. |



不以 // 开头的 target pattern，以相对于 working 目录的方式解析，下述例子假设有一个 working 目录：foo:

| `:foo`        | Equivalent to `//foo:foo`.                                   |
| ------------- | ------------------------------------------------------------ |
| `bar:wiz`     | Equivalent to `//foo/bar:wiz`.                               |
| `bar/wiz`     | Equivalent to:`//foo/bar/wiz:wiz` if `foo/bar/wiz` is a package`//foo/bar:wiz` if `foo/bar` is a package`//foo:bar/wiz` otherwise |
| `bar:all`     | Equivalent to `//foo/bar:all`.                               |
| `:all`        | Equivalent to `//foo:all`.                                   |
| `...:all`     | Equivalent to `//foo/...:all`.                               |
| `...`         | Equivalent to `//foo/...:all`.                               |
| `bar/...:all` | Equivalent to `//foo/bar/...:all`.                           |



- bazel 允许 target 部分使用 / 而不是只有 :，在 bash finelname expansion 中很方便。
  - 例如 ：`foo/bar/wiz` is equivalent to `//foo/bar:wiz` (if there is a package `foo/bar`) or to `//foo:bar/wiz` (if there is a package `foo`).

- bazel 支持一次命令中有多个 target pattern
  - `bazel build foo/... bar/...`：构建 foo 目录和 bar 目录下的所有 target。
  - `bazel build -- foo/... -foo/bar/...`：构建 foo 目录下 除了 foo/bar 下的所有 targets
    - `--` 是必要的，防止后续以 `-` 开头的参数，被解释为额外的 options 
    -  这种方式并不能真正保证 /foo/bar 的 targets 不被构建，因为其target可能被其他部分依赖了，作为依赖被构建了
- target 中带有  `tags = ["manual"]`  的 target，在执行 build / test 的命令时（query 命令不做此过滤），不会被包含在 target pattern（..., :*, :all 等） 中。必须明确指定。



### 拉取外部依赖

默认情况下，bazel 在构建时下载和链接外部依赖。但我们有例外时 ：

- 想知道何时加入了新的外部依赖

- 无网前，提前拉取依赖



可以指定 `--fetch=false` flag 防止自动拉取。

- 对于本地文件系统中文件的使用，不管这个 flag 是否设置，都会去拉取。
- 当关闭此 flag，但是构建过程中又需要外部的依赖，构建则会失败。
- 当需要运行 bazel fetch 时，此 flag 需要开启。
  - fetch 发生在第一次构建前
  - 加入新的外部依赖后
- 拉取好外部依赖后， WORKSPACE 文件不修改时，fetch 不必再运行。
- `bazel fetch //foo:bar //bar:baz` 拉取这俩 target  需要的外部依赖。
- `bazel fetch //...` 拉取 workspace 需要的所有外部依赖。



#### repository cache

bazel 试图避免多次拉取同一个文件，将所有下载的文件缓存在 `~/.cache/bazel/_bazel_$USER/cache/repos/v1/`. （此位置可以通过  `--repository_cache` option 修改 ），供所有 workspaces 和 installed versions 共享。

- 如果下载请求的文件有 sha256，而 cache 中有文件有相同的 sha256，则可以直接使用，不仅安全，且避免非必要下载。
- 每次命中缓存，cache 中的文件的修改时间都被更新，最后一次使用 cache 目录中文件的时间很容易确定。for example to manually clean up the cache. The cache is never cleaned up automatically, as it might contain a copy of a file that is no longer available upstream.



#### Distribution files directories

和 repository cache 类似，用于避免重复下载。两者的主要差别是distribution directory  需要人工准备。

bazel 在搜索 repository cache之前，先在 分布式目录中搜寻。

使用  [`--distdir=/path/to-directory`](https://bazel.build/reference/command-line-reference#flag--distdir) option，可以指定额外的目录去寻找文件，指定目录下的文件都可用（可能使得目录很大）。如果 WORKSPACE 文件中指定了sha256，那么只有匹配的文件会被用。



#### 隔离环境中运行 bazel

为了保证 bazel 二进制的尺寸较小，Bazel 的隐式依赖在首次运行时，通过网络连接拉取。这些隐式的依赖包含的 toolchains 和 rules 可能并不是对所有人都是有必要的。但在隔离环境中会有问题，尽管可能在

- 可以在有网的情况下，准备一个包含这些依赖的 distribution directory，通过线下的方式传输到隔离环境中。

- 对于每个新的 Bazel 二进制，都需要准备这样的目录，因为每个 release 的隐式依赖，可能会不同。
- 构建依赖，需要拉取 bazel 的仓库代码，并构建 @additional_distfiles//:archives.tar target，将生成的产物，解压到新的目录中。
  - `tar xvf bazel-bin/external/additional_distfiles/archives.tar -C "$NEW_DIRECTORY" --strip-components=3`
- 最后，在隔离环境中使用 bazel 时，`--distdir` flag 指向目录，或者在 .bazelrc 里面新增一条 `build --distdir=path/to/directory`



### Build configurations and cross-compilation



## commands 和 options

--copts 等



## 编写 bazelrc 文件



## 通过脚本调用 Bazel



## 客户端和服务端架构

Bazel 系统是 long-lived server 进程。相比 batch-oriented  （A technique that uses a single program loading to process many individual jobs, tasks, or requests for service.） 实现，可以做诸多优化，如缓存 BUILD 文件，依赖图等，从而加速增量的构建，允许不同的命令共享相同的 packages 缓存，使得 queries 也很快。

- 当运行 bazel 时，启动了一个 client。
- client 基于output base（默认是 base workspace 目录和 uderid）寻找 server。base workspace 和 urserid 的不同使得并发执行成为可能，因为会启动不同的server。
  - 如果同一个用户在多个 workspace 中构建，则有多个 Bazel server 进程。
  - 多个用户可以在同一个 workstation 并行构建，因为基于不同的 userids，output base 不同。
  - 如果client 找不到运行中的 server 实例，则新建一个。
  - server 进程空闲一段时间（默认 3h）后会停止。可以通过  startup option `--max_idle_secs`) 修改这一选项。
    - 不要同时留下太多空闲的 server，可以在执行完成后显示地关闭，或者设置小一点的过期时间。
- 使用 ps x/ ps -e f，Bazel server 进程的名字是 bazel(dirname)。
  - dirname 是包含 workspace 目录根的目录。
  - 如果用 ps 的其他选项，server 进程名可能只显示 java。
  - 可以使用 shutdown 命令停止进程：
    - 停止前会先检查任务是否完成
    - 一般没什么用，但是在脚本中很有用，当知道某个构建在特定的workspace 不会再发生。
    - 接受  `--iff_heap_size_greater_than _n_`  option，需要输入一个整形的参数（MB）则根据已用内存量设定关闭。
  - 运行 `bazel` 时，client 首先检查 server 是否是适当的版本；
    - 停止旧的 server，启动新 server。
    - 确保使用长时间运行的 server 进程，不会影响正确的版本。





# 配置构建

## 可配置的属性



## 与 c++ 规则集成





## Toolchain Resolution Implementation Details？no need





## 代码覆盖率





使用 Starlark （domain-specific language：DSL）声明编译目标

编译目标：具体说明了编译的 输入和依赖，编译规则，以及编译规则的可配置选项

编译规则：具体说明了Bazel 使用的编译工具（如编译器和链接方法及对应的配置）

Starlark

语法和python3类似，支持 None，bool，dict，function，int，list，string，文件为.bzl 后缀。

偏好不变性，只有 lists 和 dicts 是可变数据结构，但是对可变数据的修改只对当前上下文下创建的对象有效。

因为Bazel 编译是并行执行，当前文件定义的 list 一旦脱离当前环境，比如被其他文件饮用，则无法再编辑修改。



### BUILD 和 .bzl 文件的区别

BUILD 文件通过调用规则注册目标；.bal 文件提供了常量，规则，宏和函数的定义。

原生函数（native functions）和规则（native rules）在 BUILD 文件中是全局符号；而.bal 文件需要通过 native 模块加载这些函数和规则。

BUILD 文件中的限制：1）不可声明函数；2）不可使用*args和**kwarg参数。

BUILD 文件

大部分 BUILD 文件中只包含编译 rule 的声明，所以顺序是无关的。

BUILD 中不能包含函数的定义（为了将 code 和 data 分离，但是list comprehension 和 if 表达式是可以的）。函数可以在 .bzl 文件中声明；不允许有 *args 和 **kwargs 参数，需要显示列出所有的参数。

Starlark 的程序不可以执行任意的IO，因此对 BUILD 文件的解释不受外界影响（只依赖已知的输入，因此输出是可复制的）。

load



Bazel 的扩展是以 .bzl 结尾的文件。通过 load 声明从扩展中导入符号。
load("//foo/bar:file.bzl", "some_library") ：加载 foo/bar/file.bzl 并将符号 some_library 添加到环境中。这样可以使用新的 rule，function，或者常量（如 string，list 等）。
● load 必须在最外层，不可以写在 function 内。
● load 的第一个参数表示 .bzl 文件，如果是一个相对的 label，则根据包含当前 bzl 文件的 package 来解析，而不是目录，且此 label 用前置 ：来表示。
● load 支持别名，可以给导入的符号分配不同的名字。load("//foo/bar:file.bzl", library_alias = "some_library") 
● 可以在 load 中同时定义别名和符号：load(":my_rules.bzl", "some_rule", nice_alias = "some_other_rule") 

.bzl 文件中，对于_开头的符号不会被导入，并且不能从其他文件中加载，目前，可见性不影响加载。

编译 rule 的类型
● *_binary：编译可执行程序
● *_test：可依赖其他 libraries。
● *_library：指定单独编译的模块。可以依赖其他 libraries，binaries。



