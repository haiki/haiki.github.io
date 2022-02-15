---
title: sed
date: 2022-01-10 09:44:13
categories:
- 工具
tags:
- linux
---
@[toc]

shell脚本最常见的一个用途就是**处理文本文件。检查日志文件、读取配置文件、处理数据元素**，shell脚本可以帮助我们将文本文件中各种数据的日常处理任务自动化。但仅靠shell脚本命令来处理文本文件的内容有点勉为其难。

如果<u>想在shell脚本中处理任何类型的数据</u>，你得熟悉Linux中的sed和gawk工具。这两个工具能够极大简化需要进行的数据处理任务。

# sed 编辑器简介
- **sed编辑器**被称作流编辑器 （stream editor），流编辑器会在编辑器处理数据之前基于预先提供的一组规则来编辑数据流。
- 在交互式文本编辑器中（比如vim），你可以用键盘命令来交互式地插入、删除或替换数据中的文本。

sed编辑器可以**根据命令来处理数据流中的数据**，这些命令要么从命令行中输入，要么存储在一个命令文本文件中。在流编辑器**将所有命令与一行数据匹配完毕后，它会读取下一行数据并重复这个过程。**在流编辑器处理完流中的所有数据行后，它就会终止。

sed编辑器会执行下列操作：
1. 一次从输入中读取一行数据。
2. 根据所提供的编辑器命令匹配数据。
3. 按照命令修改流中的数据。
4. 将新的数据输出到STDOUT 。

## sed 命令

格式：`sed options script file`
- options 选项允许你修改sed 命令的行为
  - `-e script`：在处理输入时，将script 中指定的命令添加到已有的命令中
  - `-f file`：在处理输入时，将file 中指定的命令添加到已有的命令中
  - `-n`：不产生命令输出，使用print 命令来完成输出
  - `-i`：就地修改
- script 参数指定了应用于流数据上的单个命令。如果需要用多个命令，要么使用 `-e` 选项在命令行中指定，要么使用 `-f` 选项在单独的文件中指定。
- file 参数表示要处理的数据文件？


1. 在命令行定义编辑器命令：`$ echo "This is a test" | sed 's/test/big test/'`
   - 默认情况下，sed编辑器会将指定的命令应用到STDIN 输入流上。这样你可以直接将数据通过管道输入sed编辑器处理。
   - sed编辑器并不会修改文本文件的数据。它只会将修改后的数据发送到STDOUT。

2. 在命令行使用多个编辑器命令：`$ sed -e 's/brown/green/; s/dog/cat/' data1.txt`
   - 命令之间必须用分号隔开，并且在命令末尾和分号之间不能有空格。
   - 也可以用bash shell中的次提示符来分隔命令。只要输入第一个**单引号标示出sed程序脚本的起始**（sed编辑器命令列表），bash会继续提示你输入更多命令，直到输入了标示结束的单引号。要在封尾单引号所在行结束命令。bash shell一旦发现了封尾的单引号，就会执行命令。
     ```
     $ sed -e '
     > s/brown/green/
     > s/fox/elephant/
     > s/dog/cat/' data1.txt
     ```

3. 从文件中读取编辑器命令：`$ sed -f script1.sed data1.txt`

# script

## `s （substitute）命令`
用斜线间指定的第二个文本字符串来替换第一个文本字符串模式。

### 替换标记
- 默认情况下它只替换每行中出现的第一处。要让替换命令能够替换一行中不同地方出现的文本必须使用替换标记 （substitution flag）。替换标记会在替换命令字符串之后设置。`s/pattern/replacement/flags`
 - 数字，表明新文本将替换第几处模式匹配的地方；
 - g ，表明新文本将会替换所有匹配的文本；
 - p ，表明匹配所在行的内容要打印出来；和 -n 搭配使用，只输出被替换命令修改过的行。
 - w file ，将替换的结果写到文件中。`sed 's/test/trial/w test.txt' data5.txt`

### 替换字符
- 正斜线通常用作字符串分隔符，因而如果它出现在了模式文本中的话，必须用反斜线来转义。`$ sed 's/\/bin\/bash/\/bin\/csh/' /etc/passwd`
- 要解决这个问题，sed编辑器允许选择其他字符来作为替换命令中的字符串分隔符：`$ sed 's!/bin/bash!/bin/csh!' /etc/passwd`。感叹号被用作字符串分隔符，这样路径名就更容易阅读和理解了。


## 使用地址
默认情况下，在sed编辑器中使用的命令会作用于文本数据的所有行。如果只想将命令作用于特定行或某些行，则必须用行寻址 （line addressing）。

### 以数字形式表示行区间

1. 在命令中指定的地址可以是单个行号：`$ sed '2s/dog/cat/' data1.txt`

2. 用起始行号、逗号以及结尾行号指定的一定区间范围内的行：`$ sed '2,3s/dog/cat/' data1.txt`
   - 如果想将命令作用到文本中从某行开始的所有行，可以用特殊地址——**美元符**。

**在单行上执行多条命令，可以用花括号将多条命令组合在一起**。sed编辑器会处理地址行处列出的每条命令。也可以置地址区间。
```
$ sed '2{
> s/fox/elephant/
> s/dog/cat/
> }' data1.txt
```

#### 用文本模式来过滤出行
sed编辑器允许指定文本模式来过滤出命令要作用的行。`/pattern/command`，例如 `$ sed '/Samantha/s/bash/csh/' /etc/passwd`

## 删除行

与替换命令使用方式类似，它会删除匹配指定寻址模式的所有行。使用该命令时要特别小心，如果你忘记加入寻址模式的话，流中的所有文本行都会被删除。
```
$ sed 'd' data1.txt  // 删除所有行
$ sed '3d' data1.txt  // 删除指定的第二行
$ sed '2,3d' data1.txt  // 删除区间 2-3 行
$ sed '/pattern/d' data1.txt // 删除匹配pattern的所有行
$ sed '/pattern1/,/pattern2/d' data1.txt // 删除指定行之间的所有行，包括指定行。你指定的第一个模式会“打开”行删除功能，第二个模式会“关闭”行删除功能。
- 如果 pattern1 在 pattern2 之后再次出现，则删除功能会再次打开，删除后面的所有行。
- 如果 pattern2 不存在，同理，也会删除 pattern1 后的所有行
```
## 插入和附加文本

- `插入（insert ）命令（i ）`会在指定行前增加一个新行；
- `附加（append ）命令（a ）`会在指定行后增加一个新行。

它们不能在单个命令行上使用，比如只插入一行时，可以在一条命令行上操作；但是当插入两行时，就得以两行的形式表示，且新行最后用反斜线。
```
sed '[address]command\
new line\
new line\
new line' 
```
你必须指定是要将行插入还是附加到另一行。可以匹配一个数字行号或文本模式，但不能用地址区间。这合乎逻辑，因为你只能将文本插入或附加到单个行的前面或后面，而不是行区间的前面或后面。
如果你有一个多行数据流，想要将新行附加到数据流的末尾，只要用代表数据最后一行的美元符就可以了：`$ sed '$a\line' data.txt`

## 修改行
与插入和附加文本类似
```
$ sed '3c\
> This is a changed line of text.' data6.txt

// 用 pattern
$ sed '/pattern/c\
> This is a changed line of text.' data6.txt

// 用区间寻址会替换这两行
$ sed '2,3c\
> This is a new line of text.' data6.txt
```

## 转换
`[address]y/inchars/outchars/`
- 转换（transform ）命令（y ）对inchars 和outchars 值进行一对一的映射。inchars 中的第一个字符会被转换为outchars 中的第一个字符，第二个字符会被转换成outchars 中的第二个字符。这个映射过程会一直持续到处理完指定字符。如果inchars 和outchars 的长度不同，则sed编辑器会产生一条错误消息。
- 转换命令是一个全局命令，也就是说，它会文本行中找到的所有指定字符自动进行转换，而不会考虑它们出现的位置。同理，你无法限定只转换在特定地方出现的字符。
```
$ echo "This 1 is 2 is 3 hah" | sed 'y/123/456/'
This 4 is 5 is 6 hah
$
```

## 打印输出
### p 命令
```
$ echo "this is a test" | sed 'p‘ // 单纯打印文本
$ sed -n '/number 3/p' data6.txt // 打印匹配到 number 3 的行，txt文本内容全部都会打印，但是匹配行会出现两次
$ sed -n '2,3p' data6.txt // 区间行打印多次
```
### = 命令：打印行号
```
$ sed -n '/number 4/{
> =
> p
> }' data6.txt
4
This is line number 4.
```

### 列出（list ）命令（l ）
打印数据流中的文本和不可打印的ASCII字符。“任何不可打印字符要么在其八进制值前加一个反斜线，要么使用标准C风格的命名法（用于常见的不可打印字符），比如\t ，来代表制表符。
```
$ cat data9.txt
This    line    contains        tabs.
$
$ sed -n 'l' data9.txt
This\tline\tcontains\ttabs.$
$
```

## 使用sed处理文件
### w 命令：向文件中写入行
`[address]w filename`
- filename 可以使用相对路径或绝对路径，但不管是哪种，运行sed编辑器的用户都必须有文件的写权限。
- 地址可以是sed中支持的任意类型的寻址方式，例如单个行号、文本模式、行区间或文本模式。
- src 是命令之外的file，dst 是 filename，即把 file 的内容写入 filename。命令里指定的行和位置针对的是 src 源文件。

如果要根据一些公用的文本值从主文件中创建一份数据文件，比如下面的邮件列表中的，那么w 命令会非常好用。
```
$ cat data11.txt
Blum, R       Browncoat
McGuiness, A  Alliance
Bresnahan, C  Browncoat
Harken, C     Alliance
$
$ sed -n '/Browncoat/w Browncoats.txt' data11.txt // sed编辑器会只将包含文本模式的数据行写入目标文件。

$
$ cat Browncoats.txt
Blum, R       Browncoat
Bresnahan, C  Browncoat
$
```

### r命令：从文件中读取行
读取（read ）命令（r ）允许你将一个独立文件中的数据插入到数据流中。

读取命令的格式如下：`[address]r filename`

- filename 参数指定了数据文件的绝对路径或相对路径。你在**读取命令中使用地址区间，只能指定单独一个行号或文本模式地址。**
- sed编辑器会将文件中的所有文本行插入到指定地址后。
- 要在数据流的末尾添加文本，只需用美元符地址符就行了。
- 从filename中读取行，写入外面的 f。

读命令和删除命令配合使用：利用另一个文件中的数据来替换文件中的占位文本。
举例来说，假定你有一份套用信件保存在文本文件中：
```
$ cat notice.std
Would the following people:
LIST
please report to the ship's captain.
$
```

套用信件将**通用占位文本LIST 放在人物名单的位置**。要在占位文本后插入名单，只需读取命令就行了。但这样的话，占位文本仍然会留在输出中。要删除占位文本的话，你可以用删除命令。结果如下：

```
$ sed '/LIST/{
> r data11.txt
> d
> }' notice.std
Would the following people:
Blum, R       Browncoat
McGuiness, A  Alliance
Bresnahan, C  Browncoat
Harken, C     Alliance
please report to the ship's captain.
$
```

r 命令是：读别人，写入我；w 命令是：读我，写别人。




## 参考/引用
摘录来自: [美] Richard Blum. “Linux命令行与shell脚本编程大全（第3版）。” Apple Books. 