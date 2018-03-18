---
title: Programming FAQ - Core Language（语言核心部分）
date: 2018-03-14 19:23:43
tags: [Python, Documentation, Translation]
---

学习编程过程中官方文档真的是非常优秀的学习材料，已经被 __python.org__ 深深吸引，以下是 [Programming FAQ](https://docs.python.org/2/faq/programming.html) 目录，本文讨论的是 `Core Language` 部分:

+ General Questions
+ __Core Language [没错，就是本篇文章]__
+ Numbers and strings
+ Sequences (Tuples/Lists)
+ Dictionaries
+ Objects
+ Modules

<!-- more -->

## 给变量赋值的时候怎么就遇到了 UnboundLocalError 错误呢？

__原标题: Why am I getting an UnboundLocalError when the variable has a value?__

通过在函数体内的某个地方添加一个赋值语句来修改之前工作的代码，这时如果出现 UnboundLocalError 是一件让人感到惊讶的事情。

这段代码:

```Python
>>> x = 10
>>> def bar():
...     print x
>>> bar()
10
```

是正常 work 的, 但是这段代码:

```Python
>>> x = 10
>>> def foo():
...     print x
...     x += 1
```

出现了一个 UnboundLocalError 错误:

```Python
>>> foo()
Traceback (most recent call last):
  ...
UnboundLocalError: local variable 'x' referenced before assignment
```

这是因为，当对作用域中的变量进行赋值时，该变量将变为 __该作用域的局部变量__，并对在 __外部作用域__ 中任何具有 __相同名称的变量__ 进行屏蔽。由于 foo 中的最后一条语句 `x += 1` 对 x 进行了 __赋值__，因此解释器将其识别为 __局部变量__。 因此，当前面的 `print x` 尝试打印未初始化的局部变量时产生了错误。

在下面例子中可以通过 `global` 申明全局变量来访问作用域范围之外的变量。

```Python
>>> x = 10
>>> def foobar():
...     global x
...     print x
...     x += 1
>>> foobar()
10
```

这个显式声明是为了提醒你（不同于类和实例变量的表面上类似的情况），你实际上是在修改外部变量的值:

```Python
>>> print x
11
```

## Python 中局部变量和全局变量的使用规则是怎样的？

__原标题: What are the rules for local and global variables in Python?__

在 Python 中，如果一个变量在函数内仅是被引用（而没有被赋值）的话被认为是一个隐式的全局变量 _[注: 参考上面的 bar 函数]_ 。 如果一个变量在函数体内的任何位置被赋值，它被认为是一个局部变量，除非明确声明为全局变量。

【下面这段是在没读懂什么意思，防止误导，就先复制过来了。】

Though a bit surprising at first, a moment’s consideration explains this. On one hand, requiring `global` for assigned variables provides a bar against unintended side-effects. On the other hand, if global was required for all global references, you’d be using `global` all the time. You’d have to declare as global every reference to a built-in function or to a component of an imported module. This clutter would defeat the usefulness of the `global` declaration for identifying side-effects.


##

__原标题: Why do lambdas defined in a loop with different values all return the same result?__
## How do I share global variables across modules?

## What are the “best practices” for using import in a module?
## Why are default values shared between objects?
## How can I pass optional or keyword parameters from one function to another?
## What is the difference between arguments and parameters?
## Why did changing list ‘y’ also change list ‘x’?
## How do I write a function with output parameters (call by reference)?
## How do you make a higher order function in Python?
## How do I copy an object in Python?
## How can I find the methods or attributes of an object?
## How can my code discover the name of an object?
## What’s up with the comma operator’s precedence?
## Is there an equivalent of C’s “?:” ternary operator?
## Is it possible to write obfuscated one-liners in Python?
## Numbers and strings
