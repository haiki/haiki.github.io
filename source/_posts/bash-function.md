---
title: bash_function
date: 2022-04-23 22:21:56
categories:
- 工具
tags:
- bash
---

# 创建/使用函数

```bash
function func {
    cmds
}
# another method
func() {
    cmds
}

```
1. 函数使用必须在定义后了；函数名是唯一的
3. 函数执行退出后，用标准变量 `$?` 确定函数的退出状态码
    - 只看最后一条语句，所以无法判断中间是否执行成功
    - 函数退出后要马上用，否则就是其他语句的状态码了
        - 使用 return 命令 退出函数并返回特定的退出状态码，return 命令中可指定一个整数值（范围：0-255，返回后必须要马上取返回值。
        - 使用 exit n

5. 使用函数输出：函数用 `echo` 语句显示计算的结果，调用方可获取到此结果。如：
    ```bash
    function fun1 {
        echo $[ $val * 2 ]
    }
    result=$(fun1) # `fun1` 也可以，叫做命令替换，这里用来使用函数的输出
    ```

# 函数中使用变量

## 向函数传递参数

## 局部变量

#linux shell 字符串作变量名 间接变量引用 evel
https://blog.csdn.net/whatday/article/details/105466009
https://blog.csdn.net/whatday/article/details/105459629

#Shell 单引号'' 双引号"" 反引号`` 和$()的区别和用法
https://zhuanlan.zhihu.com/p/91663627