---
title: rpc
date: 2022-06-03 09:44:30
categories:
- 框架
tags:
---







# RPC

1. 在.proto（protocol buffer）的文件中，描述数据交换的格式
   a. developers.google.com/...
2. 使用protoc编译.proto文件，生成对应偏好语言的代码文件（比如：这个例子中，调用protoc将会创建c++的代码），代码文件实际上是一组在.proto描述好的函数接口，包括函数名和返回值/返回自定义的数据结构（c++里的类）
   a. 用protoc产生hello_world.pb.h/.cc, 将我们定义的协议转换成C++中的类。有可能你的环境中还没有安装，不过不要紧，在你执行了位于trpc-cpp项目(https://git.code.oa.com/trpc-cpp/trpc-cpp)中的./build.sh, 对整个项目进行构建后，会生成protoc.
   b. 不过在用户侧，我们并不会直接操作protoc, 而是借助trpc-cpp中trpc_proto_library, 帮助我们生成对应的文件。
   c. 待生成的文件有两组:hello_world.pb.h/.cc，hello_world.trpc.pb.h/.cc。其中hello_world.proto中的message等在hello_world.pb.h/.cc中生成，service相关的交互接口在hello_world.trpc.pb.h/.cc中生成。
3. 服务端实现了生成的代码文件的接口
4. 客户端将调用为客户端生成的接口，就像调用本地一个函数一样
5. tRPC负责处理客户端和服务端之间的数据交换
   参考：iwiki.woa.com/...
