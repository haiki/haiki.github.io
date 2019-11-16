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

### 为什么使用hexo呢？
因为跟着网上的教程，用jeklly搭建博客时，是直接在github上操作，fork 了别人的模板，后来发现别人的东西总是修改不干净，也不知道别人做了一些什么样的人性化设置，就很难受。放弃了...
看到其他同学用hexo搭建，十一国庆节放假用了一天试了一下，然后就成功了。主要还是别人的博客比较靠谱，非常感谢这些无私奉献的人~
hexo也有其问题，就是它的部署不会直接部署到github上，因此都在本地，每次都需要重新渲染，内容边多了的话，可能会非常卡。还有就是本地内容可能丢失，则最终还是需要用github来备份一下，这样也方便在不同的电脑终端工作。


#### 参考博客

> 洪卫的博客：<https://www.cnblogs.com/shwee/p/11421156.html>
>
> 叶落阔的博客及3-hexo的模板：<https://yelog.org/>

#### 博客生成与部署

- 首先生成： `hexo g `，然后在本地预览 `hexo s`，最后部署在github上`hexo d`
- 这是我的第一篇博客，为什么用 npm i hexo-deployer-git 不能生成一个放图片的文件夹呢？

#### 将博客原文同步到github

- 使用hexo，如果换了电脑怎么更新博客？ 

- [直上云霄的回答](https://www.zhihu.com/question/21193762/answer/489124966) - 知乎

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




