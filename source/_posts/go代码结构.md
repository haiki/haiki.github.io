---
title: golang 学习
categories:
- 编程语言
tags:
- golang
- 笔记
---

这个[学习](https://www.bogotobogo.com/) 网站真的很不错！
go 的教程在[这里](https://www.bogotobogo.com/GoLang/GoLang_HelloWorld.php)

# go 起源
  - Go语言有时候被描述为“C类似语言”，或者是“21世纪的C语言”。
  - Go从C语言继承了相似的表达式语法、控制流结构、基础数据类型、调用参数传值、指针等很多思想，还有C语言一直所看中的编译后机器码的运行效率以及和现有操作系统的无缝适配。
  - Go语言的家族树中还有其它的祖先，包括CSP（communication sequential processes，顺序通信进程，并行程序通过管道进行通信和同步控制），Pascal 等。
  - 同时，Go 语言也有很多创新的设计：Go语言的**切片**为动态数组提供了有效的随机存取的性能（这可能会让人联想到链表的底层的共享机制）；Go语言新发明的**defer语句**。

# go 背景
- Go项目是在Google公司维护超级复杂的几个软件系统遇到的一些问题的反思
- 只有通过简洁的设计，才能让一个系统保持稳定、安全和持续的进化。
- Go项目包括编程语言本身，附带了相关的工具和标准库，以及简洁编程哲学的宣言。
- Go语言源代码本身就包含了构建规范。
- Go 1.12中，当当前的目录或者父目录中有go.mod，且目录在 $GOPATH/src 之外，go 命令支持使用 modules。为了兼容性，如果目录在$GOPATH/src中，不论是否有 go.mod，都使用旧的 GOPATH 模式。

go module [参考1](https://www.bogotobogo.com/GoLang/GoLang_Modules_1_Creating_a_new_module.php)


# go 代码组织
- Go 程序组织在 package 中。一个 package 是同一个目录下的多个同时编译（compiled）的源文件，源文件中定义的函数，类型，变量，常量等，对且同一个 package 下的源文件是可见的。
- 一个Go repository 包含一个或多个 modules。一个 module 是多个同时发布（released）的 Go packages。典型地，一个 Go repo 包含一个 module，位于 repo 的根目录下。
  - go.mod 声明 module path：是同一个module 中所有 package 的 import path 的前缀；也指明了从哪里寻找文件并下载。
  - 如果 package 在 module 的当前目录或者子目录下，且没有自己的 go.mod 文件，则属于当前 module。
- import path 是用来导入 package 的string，即 module path + package 在module 中的子目录名
  - 标准库中的 package 不需要 module path prefix。

# go 项目结构
参见 [go standards](https://github.com/golang-standards/project-layout)
推荐直接看汉语版，有别人的一些简介：见[Go 面向包的设计和架构分层](https://github.com/danceyoung/paper-code/blob/master/package-oriented-design/packageorienteddesign.md)，同时[Go 包的组织和命名]（https://github.com/danceyoung/paper-code/blob/master/package-style-guideline/packagestyleguideline.md）也很推荐看一看。

go 会忽略以.或者_开头的文件或目录。
Go 项目中不要用 src 文件夹。因为 $GOPATH 环境变量指向当前使用的 workspace ，这个workspace 包含顶级目录：/pkg，/bin 和 /src。实际 go 项目是 /src 目录下的子目录，因此不推荐。（Go 1.11 后可以将项目放置在 GOPATH 之外，但也不推荐这种布局）

- /cmd
  - 此项目的主应用。目录名要和可执行程序的名字一致，不要在这个目录放置过多文件
  - 对于可以被其他项目 import 的代码，放置在 /pkg 目录下，如果不想让其他人重用，可放在/internal 目录下
  - main 函数 import 并 invoke 在/internal 和 /pkg 目录中的代码是很常见的
- /internal
  - 私有的应用和库代码
  - Go1.14 中，Go 编译器强制其他库或者应用不可以 import internal 目录下的代码
  - 可以在项目中的任何层级，包含一个或多个 internal 目录，在 internal 目录内又可以额外增加结构，从而分离共享（pkg）和非共享（app）的代码
- /pkg
  - 可以被外部应用使用的库代码
  - 明确告诉其他人，这个目录下的代码是可以安全使用的
- /vendor
  - App 的依赖，手动或者 go modules 维护
  - go mod vecdor 命令创建/vendor 目录
--------------
- /api
  - service app 目录
- web
  - web app 目录
  - static web assets, server side templates and SPAs.
- 通用的 app 目录
  - /configs
    - 配置文件模版或者默认的配置
  - /init
    - system init（systemd, upstart, sysv），以及处理manage/supervisor的配置
  - scripts
    - 执行各种 build，install，analysis 等操作的脚本，从而使得 Makefile 简单而小
  - /build
    - 用于打包和持续集成
    - 将cloud（AMI），容器（Docker），OS（deb，rpm，pkg）的包配置和脚本放置在/build/package目录下
    - 将 CI（travis, circle, drone）相关的配置和脚本放置在 /build/ci 目录下
  - /deployments or /deploy
    - IaaS, PaaS,templates (docker-compose, kubernetes/helm, mesos, terraform, bosh)
  - /test
    - 任意测试文件，还可以包含 /test/data自目录
- 其他目录
  - /docs
  - tools
    - 支持此项目的工具？这些 tools 可以 import /pkg和/internal 目录下的代码
  - exaamples
  - third_party
    - 外部帮助工具，forked code，和其他第三方 utilities
  - /githooks
  - /assets
    - 你项目repo的图片，logs等
  - /website
    - 如果不再github上放置项目，这里说明项目所在的网页

？ 不太懂 pkg，third_party ，tools 啥区别？

## go 文件结构
- 首先需要选择 module path，如 example.com/user/hello，进入hello，go mod init example.com/user/hello
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world.")
}
```

# go 控制语句


# go 工具
- go install：将二进制文件安装在go workspace的bin目录下【？没成功】
- go clean -i：删除workspace的bin目录下的二进制文件
- Gofmt：reformats Go source code
- golint prints out style mistakes.

