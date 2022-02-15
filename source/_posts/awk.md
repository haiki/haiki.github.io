---
title: awk
date: 2022-01-10 09:42:10
categories:
- 工具
tags:
- linux
---
@[toc]

# awk 编辑器简介
[awk、gawk、nawk、mawk的简单介绍](https://blog.csdn.net/gechangli7/article/details/51547641)

gawk程序是Unix中的原始awk程序的GNU版本。gawk程序让流编辑迈上了一个新的台阶，**它提供了一种编程语言而不只是编辑器命令**。在gawk编程语言中，你可以做下面的事情：
1. 定义变量来保存数据；
2. 使用算术和字符串操作符来处理数据；
3. 使用结构化编程概念（比如if-then 语句和循环）来为数据处理增加处理逻辑；
4. 通过提取数据文件中的数据元素，将其重新排列或格式化，生成格式化报告。”


# awk 命令
gawk程序的基本格式如下：`gawk options program file`

## options
- `-F fs`：指定行中划分数据字段的字段分隔符
- `-f file`：从指定的文件中读取程序
- `-v var=value`：定义gawk程序中的一个变量及其默认值
- `-mf N`：指定要处理的数据文件中的最大字段数
- `-mr N`：指定数据文件中的最大数据行数
- `-W keyword`：指定gawk的兼容模式或警告等级


## program file
1. gawk程序脚本用一对花括号来定义。你必须将脚本命令放到两个花括号（{} ）中。
2. 由于 **gawk 命令行**假定脚本是单个文本字符串，你还必须将脚本放到单引号中。

```
$ gawk '{print "Hello World!"}'
This is a test
Hello World!
hello
Hello World!
This is another test
Hello World!

# 要终止这个gawk程序，你必须表明数据流已经结束了。bash shell提供了一个组合键来生成EOF（End-of-File）字符。Ctrl+D组合键会在bash中产生一个EOF字符。这个组合键能够终止该gawk程序并返回到命令行界面提示符下。
```

## 使用数据字段变量
gawk的主要特性之一是其处理文本文件中数据的能力。它会**自动给一行中的每个数据元素（一列）分配一个变量**。

默认情况下，gawk会将如下变量分配给它在文本行中发现的数据字段：`$0` 代表整个文本行；`$1` 代表文本行中的第1个数据字段；`$n` 代表文本行中的第 n 个数据字段

在文本行中，每个数据字段都是通过**字段分隔符**划分的。gawk在读取一行文本时，会用预定义的字段分隔符划分每个数据字段。**gawk中默认的字段分隔符是任意的空白字符（例如空格或制表符）**。

## 在程序脚本中使用多个命令

1. 在命令行上的程序脚本中使用多条命令，只要在命令之间放个分号即可。

2. 也可以用次提示符一次一行地输入程序脚本命令。（不用分号）

## 从文件中读取程序
```
$ cat script3.gawk
{
text = "'s home directory is "
print $1 text $6
}
$
$ gawk -F: -f script3.gawk /etc/passwd”

# script3.gawk 程序脚本定义了一个变量来保存 print 命令中用到的文本字符串。注意，gawk程序在引用变量值时并未像shell脚本一样使用美元符。

```

## 在处理数据前/后运行脚本

gawk在读取数据前执行 BEGIN 关键字后指定的程序脚本。END 关键字允许你指定一个程序脚本，gawk会在读完数据后执行它。

如果在命令行里输入 BEGIN/END ，则需要将整个命令加单引号


```
$ cat script4.gawk
BEGIN {
print "The latest list of users and shells"
print " UserID \t Shell"
print "-------- \t -------"
FS=":"   # 特殊变量，是定义字段分隔符的另一种方法。这样就不用依靠脚本用户在命令行选项中定义字段分隔符了。
}

{
print $1 "     \t "  $7
}

END {
print "This concludes the listing"
}

$ gawk -f script4.gawk /etc/passwd
```




# 引用/参考
摘录来自: [美] Richard Blum. “Linux命令行与shell脚本编程大全（第3版）。” Apple Books. 