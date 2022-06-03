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
Bazel 构建 workspace 中以目录树组织的源文件。workspace 中的源文件由嵌套层级的 packages 组织，每个 packag 是一个目录，包含相关的源文件和一个BUILD 文件。BUILD 文件说明了可以从源文件编译出的输出。



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



### Packages

Repository 中主要的代码组织单元是 package。package 包含相关文件的集合，以及这些文件如何产出输出的说明。

- 包含 BUILD/BUILD.bazel 文件的目录是一个 package。
- 一个 package 中包含这个目录/子目录下的所有文件（包含 BUILD 文件的子目录除外，这样就没有文件/目录会成为两个不同 package 的成员 ）。



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
- Bazel 支持在 workspace 的根 package 下声明 target，如`//:foo`，但是尽量让这个 package 为空，这样每个 package 都有可描述的名字。



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

https://docs.bazel.build/versions/main/be/overview.html
build文件就是真正定义编译规则的文件了，每个目录下都有一个，每个源文件都要在BUILD中定义它的编译规则。
在cc_binary中定义可执行程序的编译规则，在cc_library中定义库的编译规则（即.o文件），在trpc_proto_library中定义pb文件的编译规则。



### 编写 BUILD 文件

● 使用 Starlark （domain-specific language：DSL）声明编译目标
● 编译目标：具体说明了编译的 输入和依赖，编译规则，以及编译规则的可配置选项
● 编译规则：具体说明了Bazel 使用的编译工具（如编译器和链接方法及对应的配置）

Starlark
● 语法和python3类似，支持 None，bool，dict，function，int，list，string，文件为.bzl 后缀。
● 偏好不变性，只有 lists 和 dicts 是可变数据结构，但是对可变数据的修改只对当前上下文下创建的对象有效。
	○ 因为Bazel 编译是并行执行，当前文件定义的 list 一旦脱离当前环境，比如被其他文件饮用，则无法再编辑修改。



### BUILD 和 .bzl 文件的区别

● BUILD 文件通过调用规则注册目标；.bal 文件提供了常量，规则，宏和函数的定义。
● 原生函数（native functions）和规则（native rules）在 BUILD 文件中是全局符号；而.bal 文件需要通过 native 模块加载这些函数和规则。
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


## 依赖

如果 A 在编译或者执行时，需要 B，那么 target A 依赖 target B。依赖图是DAG，直接依赖的是一跳，可传递依赖。
编译时，存在两种依赖图：实际（ actual ）依赖，声明（declared）依赖。

target X 实际依赖 target Y，当切仅当为了 X  编译成功， Y 必须存在，builr 且是最新的。“built” 代表：generated, processed, compiled, linked, archived, compressed, executed等在构建过程中经常发生的其他类型的任务。
target X 声明依赖 target Y，当且仅当 package X 中，从 X 到 Y 有一条依赖边。
为了正确编译，实际依赖的图 A 必须是声明依赖图 B 的子图 。也就是说，A 中直接相连的节点 x--> y. 在 D 中也必须直接相连，称作  D 是 A 的overapproximation。
overapproximation不会太过度，否则冗余的声明依赖会使得编译很慢，二进制文件很大。
因此，写 BUILD 文件时，每条 rule 都必须显示声明所有实际的直接依赖。不要试图列出所有的非直接依赖。
很重要的规则，否则可能会因为之前的操作，或者传递声明以来等原因，发生未知错误。



### 依赖的类型

所有 rule 都通用的属性 ：https://docs.bazel.build/versions/main/be/common-definitions.html
● src：
● deps
● data：测试时候输入文件等

使用 label 引用目录时，以/. 或者 / 结尾的方式很不可取，如 data = ["//data/regression:unittest/."],data = ["testdata/."], data = ["testdata/"] 

当执行重新编译时，对于这种直接包含整个目录的方式，编译系统只检查目录有没有变化（是否有增删文件），不会探测对单个文件的内部修改。因此，不要直接把目录作为编译系统的输入，应该将目录内的文件列举出来：明确说明，或者用 glob() 函数（用**强制glob函数递归），如 data = glob(["testdata/**"])  # use this instead 

directory label 只对 data 依赖有效。

Bazel 中 funciton 的BUILD 百科
Rules
推荐的 rules ：https://docs.bazel.build/versions/main/rules.html#recommended-rules
Bazel binary 中配备的 native rule 不需要用 load 声明，native rules 对于 BUILD 文件是全局可用的，可在 .bzl 文件的 native 模块找到。





## Visibility





## Platforms





## Hermeticity







## cc_library

头文件的包含检查。=》只适用于直接包含。
● 在编译中用到的头文件，必须在cc_* rule 的 hdrs 或者 srcs 中被声明。=》强制
● 对于 cc_library rules，hdrs 中的头文件，组成了这个 library 的公共接口
	○ 此 library 的 hdrs 和 srcs 中的文件可直接包含 hrds 中的头文件
	○ 其他 library 的 deps 如果依赖此 library，其 srcs 和 hdrs 也能包含此 library 的hdrs中的头文件
● srcs 中的头文件只能被此 library 的 hdrs 和 srcs 中的文件包含。
● 将文件放在 srcs 或者 hdrs，需要考虑，你是否希望此 library 的用户可以直接包含这个头文件，类似编程语言中的 public 和 private。
cc_binary 和 cc_test rule 没有对外的接口，因此没有 hdrs 属性，直接放在 srcs 中就行。
对.cc文件的编译，可以递归地包含在递归的 deps 必报中的所有 cc_library 中的 hdrs 和 srcs 中的文件。





# Output Directory Layout
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







