---
title: bash_function
date: 2022-04-23 22:21:56
categories:
- 工具
tags:
- bash
---

# 创建函数

```bash
function func {
    cmds
}
#
func() {
    cmds
}

```
1. 函数使用必须在定义后
2. 函数名是唯一的
3. 函数执行退出后，用标准变量 `$?` 确定函数的退出状态码（只看最后一条语句，所以无法判断中间是否执行成功）
4. 使用 return 命令 退出函数并返回特定的退出状态码，return 命令中可指定一个整数值（范围：0-255，返回后必须要马上取返回值。
    - 或者使用 exit n
5. 使用函数输出：函数用 `echo` 语句显示计算的结果，调用方可获取到此结果。如：

```bash
function fun1 {
    echo $[ $val * 2 ]
}
result=$(fun1) # `fun1` 也可以，叫做命令替换
```