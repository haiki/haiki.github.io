---
title: IDEA的使用：提升看源码效率
date: 2019-10-09 20:21:52
categories: 
- 工具
tags:
- IDEA
- java
---

### IntelliJ IDEA 常见文件类型的图标介绍

参见博客: [IntelliJ IDEA 常见文件类型的图标介绍](https://blog.csdn.net/qq_35246620/article/details/64157559)

### IDEA中配置参数，运行main函数

对于java程序，编译（build）指令： `javac`，运行（run）指令： `java` ，需要安装 JDK（这里又有Oracle JDK 和 OpenJDK，自行百度区别和使用），JRE。

对于windows 配置 `JAVA_HOME`，`PATH`，`classpath` 的方法见：[Java中JAVA_HOME, PATH,CLASSPATH的作用和配置值](https://blog.csdn.net/haiki66/article/details/88758199)

IDEA下可以有两种运行程序的方法：

1. 最上方工具栏分别找到 `Build`  和 `Run`对程序进行编译，运行。使用这种方法时，如果main函数运行时需要传入参数，参数的配置方法为：鼠标指向最上面工具栏的：`Run` ，选中左键进入`Edit configurations`。如下图所示。其中`program argument1`处依次填入需要的参数。

- 这里我们可以留意到的是：`Main class` 的书写形式， main 函数所在的类，是以包名的形式呈现的。那么对于不同类中的 main 函数，通过修改 类名即可。这样简单的配置之后，我们就可以直接用 `Build` 和 `Run` 来处理程序啦~ 很多命令的处理过程，都由IDE进行

![IDEA.JPG](https://i.loli.net/2019/10/09/oLgrNqhYudD5pbT.jpg)

2. 在终端（terminal）下运行输入 `javac` 和 `java` 编译运行程序（也就是IDE帮我们做的一些命令）。
   - `javac -classpath <lib> -sourcepath <src> -d <bin>`
   - `java -classpath <lib><:/;><bin> <main函数所在类>`
   - 其中，凡是用 `<>`框起来的都是要根据自己的实际情况进行更改的。
     - `-classpath`也可以简写为 `-cp`，是用来加载外部包的，比如你写的项目里面，用到了一些jar 包，那么需要把这些jar包的路径告诉 javac 这个可执行程序，也就对应于 `<lib>`，比如我的这个项目，某些 jar 包的路径是 `./lib/`，表示当前项目的当前文件夹下的lib文件夹下。
     - `-sourcepath` 需要输入的是当前项目所在的文件夹，查看当前项目所在路径可以试错，也可以自行百度。示例：`./src/`
     - `-d` 是指：编译之后生成的 `.class` 文件放在哪个文件夹下，这个的文件路径可以自己设置，一般在当前项目下，新建一个文件夹，如  `./bin` 或者 `./out` 。我们要知道的是，java 运行的都是 `.class` 文件。
     - `javac` 和 `java` 中的  `<lib><bin>` 都一样。需要注意的是，对于 `java` 命令中 <>`<:/;>` 是说，当在unix下使用此命令时，用冒号，在window下使用时，用分号。
     - `<main函数所在类>` 就是方法 1 中提到的 以包名形式的包含main函数的类名。

### IDEA中查看类继承关系

鼠标点在类名上，右键，在复选框中找到 `Diagrams` —> `show diagrams` ，则可以查看类的继承关系。

![IDEA1.png](https://i.loli.net/2019/10/10/EWPbBq7cZnfNDKe.png)

### 函数调用关系——SourceInsight

增：10月11在意外中发现了别人写的一篇博客，好仔细！推荐！

[C 源码阅读之函数(Function/Method)调用树图( Call Graph)及数据结构依赖图][http://blog.sina.com.cn/s/blog_6b6ab0890101qd9j.html]

这个有两种方式：

1. 鼠标点击需要查看的函数名，在上方的菜单栏查找一个图标，如下图所示，则会显示这个函数的调用关系。

   ![SourceInsight.png](https://i.loli.net/2019/10/10/8LRf45onVOvhYju.png)

2. 鼠标右键函数名，选择 `Show in Relation Window`，会出现和 1 一样的结果。

   1. 看下图，左下角会有一些小的选择，按照箭头标注，最左边点击一下，会出现列表形式的函数调用关系，在第一列，是 `name` ，默认情况下，函数是以字母序排序的，如果想按照函数调用顺序，可以鼠标左键点一下 `name`，会以英文形式提示你，现在是以什么排序的。类似文件夹下，以创建时间，或者修改日期等排序那种。这个选择好之后，点击第二个箭头指的，显示出的关系图就是按照你选定的函数的出现方式。（升序，降序，出现顺序）
   2.  接着就是调用关系是横着显示还是竖着显示；
   3. 再接着是设置。

![SourceInsight1.png](https://i.loli.net/2019/10/10/cZkgdznxXUWfowR.png)

![SourceInsight3.png](https://i.loli.net/2019/10/10/ZKvNI78q164dm3H.png)

还可以设置要查看的函数调用关系，是查看当前函数调用（calls）了哪些函数，调用关系显示几层。这里主要想提及的是 `View Relationship` 这里，这个选择直接决定了显示什么关系。可以看到，关系主要是用于 `Types（类型）、Functions（函数）、Variables（变量）、class（类）等` ， 需要注意的是，类的关系也可以展示，但是我更喜欢用 IDEA 的类继承关系。根据自己的需要，选择想要查看的关系图，如函数调用关系，那么要设置的是 `functions`，`Calls`表示查看这个函数开始调用了哪些，`calls and callers` 则表示此函数的调用 + 对此函数的调用。`reference ` 表示引用。

![SourceInsight4.png](https://i.loli.net/2019/10/10/NibgmIKpLftDQcG.png)

### IDEA中 显示函数的用法

`Alt + F7`，下面会显示出哪里使用了这个函数。

### IDEA中 类中的函数及变量等概览

`Alt + 7`

















