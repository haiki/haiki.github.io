---
title: my first article
date: 2019-10-03 16:03:38
top: 1
mathjax: true
categories:
- 工具
tags:
- hexo
- problem
---

### 为什么使用 hexo 框架搭建个人博客？
&emsp; 正如“点击头像”后的 Why Blog 中所说，想拥有一个可以做个人学习成长记录的博客网站。

&emsp; 跟着网上的教程，用 jeklly 搭建博客时，是直接在 github 上操作，fork 了别人的模板，因为懂的太少，发现别人的 demo 总是修改不干净，也不知道别人做了一些什么样的人性化设置，就很难受。放弃了...看到了hexo搭建教程，十一国庆节放假用了一天试了一下，成功！
ps：还是要找靠谱的博客，非常感谢开源和分享知识的大家！

### 当前效果是如何展示的？
- 本博客使用了 Hexo，它是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

- 本博客使用了[叶落阔提供的 hexo theme](https://github.com/yelog/hexo-theme-3-hexo)（即 3-hexo），来渲染网页内容。themes 相当于整个框架中可插拔的配置。

- 配置都写在 _config.yml 中。hexo init 后的文件夹称作 hexo 项目的根目录；各个 theme 目录下也有一个当前主题的 _config.yml。

- 参考 [洪卫的博客](https://www.cnblogs.com/shwee/p/11421156.html)： 1. 安装必要的软件，如 git, npm, nodejs, hexo 等；2. 生成 hexo 框架的必要代码；3. 启动本地服务器查看效果。此时的效果是 themes 文件夹下默认的 landscape 展示。

- 参考[开源的 3-hexo](https://github.com/yelog/hexo-theme-3-hexo)，将 3-hexo clone 到 themes 文件夹下，再次启动本地服务器查看效果。此时的效果都是demo，里面涉及了很多原作者的个人配置，如用户名等。

- 参考[3-hexo 的使用说明](http://yelog.org/2017/03/23/3-hexo-instruction/) 配置 hexo 根目录& theme 下的 _config.yml。可以写简单的demo.md 去看看各种配置的效果。
- 参考[洪卫的博客](https://www.cnblogs.com/shwee/p/11421156.html)，将 hexo 部署到GitHub。就可以在xx.github.io 上看到了 用github pages 托管的网页。
- 使用 hexo d 上传部署到 github 的其实是 hexo 编译后的文件，是用来生成网页的，不包含源文件。也就是上传的是在本地目录里自动生成的.deploy_git里面。但是为了备份源文件（即各种.md文档）以及多终端协同使用笔记，参考[直上云霄的回答](https://www.zhihu.com/question/21193762/answer/489124966)，用一个新的分支管理源文件，做备份。而 hexo 项目的根目录下的 _config.yml 中 deploy 配置的分支是master。因此 hexo d 时 master 分支会自动变。要同步源文件分支需要手动git
 add 等进行管理。




### 博客生成与部署

- 首先生成： `hexo g `，然后在本地预览 `hexo s`，最后部署在github上`hexo d`
- 这是我的第一篇博客，为什么用 npm i hexo-deployer-git 不能生成一个放图片的文件夹呢？

#### 功能测试
- 对于一些配置，需要在最上面标识，包括 categories，tags等
	- 比如公式的使用，在每篇文档开头设置mathjax： $v_i$

```C++
#include<iostream>
using namespace std;
int main(){
	return 0;
}
```


> 从小的 demo 开始，一步步累加。


