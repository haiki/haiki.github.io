---
title: generic_algorithm
date: 2022-02-19 11:01:59
categories:
- CPP
tags:
---

# 简介

头文件：algorithm，numeric（数值泛型算法，累加之类的数值运算？）


- 泛型算法（generic algorithm）
    - 算法：实现了经典算法的公共接口，如排序，搜索
    - 泛型的：可以用于不同类型的元素和多种容器类型（如标准库类型 vector，list，内置数组类型，or 其他）


- 一般情况下，算法不直接操作容器，运行在迭代器上，执行迭代器的操作
  - 迭代器令算法不依赖于容器类型 -> 迭代器操作元素 
  - **算法依赖元素类型上可发生的操作** -> 比如比较大小，内置元素类型有 >，< ，自定义类型需要自定义操作运算符，从而让算法可操作
  - 编程假设：**算法**不改变底层容器的大小
    - 可读，修改，移动元素
    - 由**插入迭代器**，完成向容器中添加元素


- 一般情况下，标准库算法对一个范围内的元素进行操作，此范围称作“输入范围”，由两个参数表示：指向要处理的第一个元素和尾元素之后的迭代器


# 初识

算法遍历输入范围的方式相同，但是使用范围中元素的方式不同。
## 只读算法
- find
- count
- accumulate(beg,end,init_val)
  - init_val 决定了函数使用哪个加法运算符以及返回值的类型
  - 编程假设：将元素类型加到和的类型上的操作必须是可行的
  - 自定义对象操作吗？ 和 运算符时怎么用？只要类定义了操作就可以了吧？
- equal：用于确定两个序列是否有相同的值
  - 只要两个序列的元素可以用==比较即可
  - 假设：第二个序列至少和第一个序列一样长


## 写容器元素的算法
1. 被写入的序列大小，不小于要写入的元素个数（空vector不可以直接fill，除非用了back_inserter）
2. 不改变容器的大小

- fill(beg,end,val)
- fill_n(dest,n,val)
  - 假定 dest 指向一个元素，从 dest开始的序列至少包含 n 个
- back_inserter
  - fill_n(back_inserter(vec),10,0);
- copy
- replace
- replace_copy
  - 不改变原序列

## 重排容器元素
- sort
- stable_sort：维持相等元素的原有顺序
- uniq
  - 返回的迭代器指向最后一个不重复元素之后的位置，此位置之后的元素依然存在，但是我们不知道他的值是什么。


# 定制操作

1. 希望的操作方式与默认的不同，如sort默认用升序，想用降序
2. 我们的序列保存了未定义运算符的元素类型（如 SaleData 自定义数据结构）

## 向算法传递函数

- 谓词（predicate）：可调用的表达式，其返回结果是可用于做条件的值。
  - 一元谓词（unary）：只接受单一参数
  - 二元谓词（binary）：两个参数

算法接受谓词作为第三个参数，对序列中的元素调用谓词，因此元素类型必须能转换为谓词的参数类型


