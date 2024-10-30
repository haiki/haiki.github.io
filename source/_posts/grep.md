---
title: grep
date: 2022-01-06 09:22:20
categories:
- 工具
tags:
- linux
- shell
---

# grep：输出匹配模式的行

grep 命令的命令行格式如下： `grep [option...] [patterns] [file...]`
- grep 命令会在**输入**或**指定的文件中**查找包含匹配<u>指定模式</u>的字符的行。grep 的输出就是包含了匹配模式的行。无法在文本中匹配 newline 字符。
- option 和 file 参数可以有 0/多个。patterns 参数包含 1/多个以新行分割的模式，当 options 是 `-e patterns` 或 `-f file`时，新行被忽略。
- file 为 - 时，表示标准输出；如果没有指定 input，grep 搜索这个工作目录。

默认情况下，grep 命令用基本的Unix风格正则表达式来匹配模式。Unix风格正则表达式采用特殊字符来定义怎样查找匹配的模式。

常用下面四个，具体见 options 部分
- `-v`
- `-n`
- `-c`
- `-e`

## options

### grep 程序的信息
- `--help`
- `-V` / `--version`

### 匹配控制
- `-e patterns` / `--regexp=patterns` ：匹配某些模式中的一个或多个
- `-f file` / `--file=file`：匹配文件中的模式
- `-i` / `-y` \ `--ignore-case`：忽略文件和匹配中的大小写
- `--no-ignore-case`：不忽略大小写
- `-v` / `--invert-match`：不匹配的行
- `-w` / `--word-regexp`：匹配包含整个单词的行
- `-x` / `--line-regexp`：匹配包含整个行的行

### 输出控制
- `-c` / `--count`：不输出具体行，输出行数
- `--color[=WHEN]` / `--colour[=WHEN]`：对选中的数据高亮
- `-L` / `--files-without-match`：输出匹配行所在文件的名字，匹配到第一次出现就返回
- `-m num` / `--max-count=num`：只搜索前 m 行
- `-o` / `--only-matching`：只输出匹配行的匹配的部分，每个匹配一行输出
- `-q` / `--quiet / --silent`：不向标准输出输出。如果搜索到匹配，马上返回 0 状态码，如果有错误，也马上返回。同 -s / --no-messages。

### 输出行前缀控制
- `-b` / `--byte-offset`
- `-H` / `--with-filename`：输出匹配时，前缀输出文件名
- `-h` / `--no-filename`：当输入只有一个文件时，不输出文件名
- `--label=LABEL` 
- `-n` / `--line-number`：输出匹配所在的行数
- `-T` / `--initial-tab`：输出前缀时，以tab形式展示
-` -Z` / `--null`

### 上下文行的控制
Context lines 是接近匹配行的非匹配行。
- `-A num` \ `--after-context=num`：输出匹配行后 num 行
- `-B num` \ `--before-context=num`：输出匹配行前 num 行
- `-C num` \ `-num \ --context=num`：输出匹配行前，后 num 行
- `--group-separator=string`
-` --no-group-separator`


“alias grep='grep --color=auto”




有四种 grep 的变种，通过 options 控制：
- `-G` / `--basic-regexp`：默认的（BRE：basic regular expressions）
- `-E` / `--extended-regexp`：支持POSI，将匹配模式解释为扩展的正则匹配表达式 （EREs）
- `-F` / `--fixed-strings`：egrep 命令支持POSIX扩展正则表达式。POSIX扩展正则表达式含有更多的可以用来指定匹配模式的字符。
- `-P` / `--prel-regexp` ：fgrep 支持将匹配模式指定为用换行符分隔的一列固定长度的字符串。这样就可以把这列字符串放到一个文件中，然后在fgrep 命令中用其在一个大型文件中搜索字符串了。

## 参考
- 部分摘录来自: [美] Richard Blum. “Linux命令行与shell脚本编程大全（第3版）。” Apple Books.
- 部分翻译自 `info grep`