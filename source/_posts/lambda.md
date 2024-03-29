---
title: lambda
date: 2022-02-15 10:07:26
categories:
- CPP
tags:
---

# 背景介绍

算法可接受一元谓词/二元谓词，传递给算法的函数必须只有1/2个参数。局限性：算法只支持1个/2参数，而需要传入2/3+个参数。需要其他特性满足需求。

可向算法传递任何类别的可调用对象。

- 可调用对象（callable object）：对于一个对象或者表达式，如果可以**对其使用调用运算符**，即()，则称为可调用的。
  - 谓词是可调用表达式
  - 函数
  - 函数指针
  - 重载了函数调用运算符的类
  - lambda 表达式
    - 表示可调用的代码单元，可理解为未命名的内联函数
    - 与函数类似，不同在于：可能定义在函数内部

# lambda 

格式 `[capture list] (parameter list) -> return type { function body }`
- [capture list] lambda 所在函数中定义的局部变量的列表
  - lambda 将局部变量包含在其捕获列表中明确表示将会使用这些变量。
  - 只有捕获了函数的局部变量，才能在lambda 的函数体中使用该变量。
  - lambda 可以直接使用局部 static 变量和他它在函数之外声明的名字。
  - 值捕获：
    - 捕获的变量必须可拷贝。
    - 函数传值参数类似，区别在于：被捕获的变量是在 lambda 创建时拷贝的，而不是调用时。
    ```cpp
    void fcn1 {
        size_t v1 = 42;
        auto f = [v1] {return v1;};
        v1 = 0;
        audo j = f(); // j = 42，保存了创建 lambda 时 v1 的拷贝
    }
    ```
    - 值被拷贝的变量，lambda 不会改变其值，如果希望改变捕获的变量，必须在参数列表首加上关键字 `mutable`。因此可变 lambda 可省略参数列表。
  - 引用捕获
    - 与返回引用的限制和问题一致：需要确保 lambda 执行时所引用（指针/迭代器指向）的对象确实存在。因为 lambda 捕获的都是局部变量，如果在函数外使用此 lambda，引用捕获原函数的局部变量已经消失了，所以有问题。
    - 函数可直接返回可调用对象，或者返回一个类对象，该类含有可调用对象的数据成员。如果函数返回了 lambda，与函数不能返回一个局部变量的引用类似，此lambda 不可以包含引用的捕获。
    - 如果可能，应该避免捕获指指针/引用。
    - 引用的变量是否可变取决于此引用指向的是否为 const 类型。
  - 显示捕获
    - 明确写出要捕获哪个变量。
  - 隐式捕获
    - 由编译器根据 lambda 体中的代码推断我们要使用哪些变量。
    - & ： 引用捕获， = ：值捕获。
  - 显示和隐式捕获可以同时使用，此时隐式捕获 &/= 作为捕获列表的第一个元素，后面的显示捕获必须使用和隐式捕获不同的方式。


- 尾置返回类型
  - 其他普通函数中，当返回类型比较复杂（如数组的指针or引用时也使用）auto f() -> int(*)[10] {}
- 必须包含：`[capture list]` 和 `{ function body }`
  - `auto f = [] {return 42;}` 只有 return 语句，返回类型从返回的表达式推断而来，
    - `{ function body }` 中包含任何其他语句，且未指定返回类型时，则默认返回 void

## lambda 是函数对象
当定义 lambda 时，编辑器会生成一个与 lambda 对应的新的（未命名）的新类型。
- 该类中有重载的函数调用运算符
- 该类中必须为每个值捕获的变量建立数据成员，并作为构造函数的参数初始化
- 对于 lambda 通过引用捕获的变化，程序负责确保 lambda 执行时所引用的对象确实存在，因此编译器可以直接使用该引用而无需存储为数据成员
- 此类中不包含默认构造函数，赋值运算符及默认的析构函数，是否包含拷贝/移动构造函数通常需要视捕获的数据成员类型而定


向一个函数传递 lambda 时，同时定义了一个新类型和该类型的一个对象，传递的参数就是此编译器生成的类类型的未命名对象。

使用 auto 定义用 lambda 初始化的变量时，定义了一个从 lambda 生成的类型的对象。


# 参数绑定

## 背景

当算法只支持一元谓词，而功能必须用 2 个参数时，可以将有2个参数的可调用对象适配为1个参数的可调用对象。

如果 lambda 的捕获列表为空，通常可以使用函数来替代它。
1. 如果需要很多地方使用相同的操作，应该定义为函数。
2. 如果一个操作要访问很多语句才能完成，使用函数更好

可将 bind 看作是通用的**函数适配器**，它接受一个可调用对象，生成一个新的可调用对象来“适应”原对象的参数列表。

- **适配器**是标准库中的通用概念，本质上是一种机制，能使某种事物的行为看起来像另一种事物一样。
  - 容器适配器：接受一种已有的容器类型，使其行为看起来像一种不同的类型
  - 顺序容器适配器：stack，queue，priority_queue

## bind 使用
头文件：`functional`

一般形式：`auto newCallable = bind(callable, arg_list);`

arg_list 是 callable 的参数。当调用 newCallable 时，newCallable 会调用 callable，并传递给 arg_list 的参数。

arg_list 中的参数可能包含形如 `_n` 的参数，`_n` 是一个参数，表示“占位符（placeholder）”，是 newCallable 的参数，n 表示在 newCallable 中参数的位置。如 `_1/_2` 是 newCallable 的第一/二个参数，依次类推。

使用占位符(`_n`)时需要引入命名空间：`using namespace std::placeholders;`。头文件 `functiondal`。

## bind 的参数
可以用 bind 重排参数顺序：
- 传给 g 的参数按位置绑定到占位符，即：第一个参数绑定到 _1，第二个参数绑定到 _2。当调用f时，按照 f 中的顺序放置。
    ```cpp
    auto g = bind(f, a, b, _2, c, _1);
    g(X, Y);
    f(a, b, Y, c, X);
    ```

绑定引用参数：
- 默认情况下，bind 中非占位符的参数被拷贝到 callable 中，但是，与 lambda 类似，有些参数 1）希望以引用的方式传递；2）要绑定的类型无法拷贝。
  ```cpp
  for_each(words.begin(), words.end(),
    [&os,c](const string& s) { os << s << c;});

  ========>

  ostream &print(ostream &os, const string& s, char c) {
      os <<s << c;
  }
  for_each(words.begin(), words.end(), bind(print, os, _1, ' ')); // 有误， ostream 是可不拷贝的

  ====> 
  for_each(words.begin(), words.end(), bind(print, std::ref(os), _1, ' '));
  ```

如果我们希望传递给 bind 一个对象而又不拷贝它，必须使用标准库 ref 函数。
- ref 函数返回一个对象，包含给定的引用，此对象是可拷贝的。
- cref 函数生成保存 const 引用的类。
- ref，cref 定义在头文件 `functional` 中。









