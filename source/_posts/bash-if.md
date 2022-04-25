---
title: bash_if
date: 2022-04-23 21:06:06
categories:
- 工具
tags:
- bash
---

# if 判断是普通命令
## if-then 语句

```bash
if cmd 
then
    cmds
fi
###
if cmd; then
    cmds
fi
```
1. 根据 cmd 命令退出状态码确定是否执行then
2. 这里的 cmd 就是 pwd，grep 之类的命令

## if-then_else 语句

```bash
if cmd 
then
    cmds
else
    cmds
fi
```

## 嵌套 if

```bash
if cmd1
then
    cmds
elif cmd2
then
    cmds
fi
```

# test 命令
if-then 语句不能测试命令退出状态码之外的条件。

```bash
if test conditions
then
    cmds
fi
```

1. test 会测试不同的条件，如果成立，则推出并返回退出状态码0
2. conditions 是test 命令要测试的一系列参数和值，当为空时，test 返回非0。可以用来测试某个变量是否有值


# [] 
[] 类似 test 命令，可以判断三类条件：
1. 数值比较：使用文本比较，用数学符号可能被解析为字符串从而得到错误的结果。（很神奇，执行数学运算，又是直接用 `$[ $val + $val2 ]` 等数学符号
    - bash 只能处理整数，如果只用 echo 显示结果没限制 `if [ $val1 -eq $val2 ]`
2. 字符串比较：使用标准的数学比较符号，如 <， >， = 等
    - 比较字符串的相等性，测试会将所有的标点和大小写考虑在内 `if [ $val1 != $val2 ]`
    - 字符串顺序
        - 大于小于号必须转义，否则会被当作转义符 `if [ $val1 \< $val2 ]`
        - 使用 ascii 顺序：大写字母比小写字母小，而 bash 的 sort 命令是按照系统本地化语言定义的排序顺序，即小写在大写前面
    - 字符串大小
        - `if [ -z $val ]` 判断 $val 的长度是否为 0，从而判断字符串是否为空，为空返回 true
        - `if [ -n $val ]` 字符串不为空则返回 true，未被定义过的也为空
        - 使用未定义/未初始化的变量不好，所以在用于数值/字符串比较之前，需要用 -n/-z 先测试下变量是否有值
3. 文件比较：测试 Linux 文件系统上文件和目录的状态
    - `-d`：file 是否存在并且是目录
    - `-e`：file 是否存在
    - `-f`：file 存在 & 是文件
    - `-r`：存在 & 可读
    - `-s`：存在 & 非空
    - `-w`：存在 & 可写
    - `-x`：存在 & 可执行
    - `-O`：存在 & 属当前用户所有
    - `-G`：存在 & 默认组与当前用户相同
    - `file1 nt file2`：检查 1 是否比 2 新
    - `file1 ot file2`：verse

# 复合条件测试
1. [ condation1 ] && [ condation2 ]
2. [ condation1 ] || [ condation2 ]

# (()) 用于数学表达式
(( expression ))：exp 可以是任意的数学赋值或表达式，不需要将表达式计算符号（>, < 之类的做转义，相比 test 命令更高级）
- val++, val--, ++val, --val, 
- ! 逻辑去反
- ~ 位取反
- ** 幂运算
- << , >>： 位移
- &, |：位布尔和/或
- &&, ||：逻辑和/或

# [[]] 用于高级字符串处理
[[ expression ]]
- 除了 test 中支持的功能外，还支持模式匹配
