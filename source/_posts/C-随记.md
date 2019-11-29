---
title: C++随记
date: 2019-11-29 09:58:19
categories:
- 编程语言
tags:
- C++
---
- 想要随机产生0-100间的整数，从而做排序的操作

  - [C++产生随机数](https://www.cnblogs.com/VVingerfly/p/5990714.html)             
  - 通常用 `srand((unsigned)time(0)) `或者`srand((unsigned)time(NULL))`来 产生种子。如果仍然觉得时间间隔太小。可以在`(unsigned)time(0)`或者`(unsigned)time(NULL)`后面乘上某个合适的整数。 例如,`srand((unsigned)time(NULL)*10)`。
  - [浮点随机数产生](https://blog.csdn.net/yangziluomu/article/details/102013793)

  