---
title: Programming FAQ - Core Language（语言核心部分）
date: 2018-03-14 19:23:43
tags: [Python, Documentation, Translation]
---

学习编程过程中官方文档真的是非常优秀的学习材料，已经被 __python.org__ 深深吸引，以下是 [Programming FAQ（原文链接）](https://docs.python.org/2/faq/programming.html) 目录，本文讨论的是 `Core Language` 部分:

+ General Questions
+ __Core Language [没错，就是本篇文章]__
+ Numbers and strings
+ Sequences (Tuples/Lists)
+ Dictionaries
+ Objects
+ Modules

<!-- more -->

## 1、给变量赋值的时候怎么就遇到了 UnboundLocalError 错误呢？

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

## 2、Python 中局部变量和全局变量的使用规则是怎样的？

__原标题: What are the rules for local and global variables in Python?__

在 Python 中，如果一个变量在函数内仅是被引用（而没有被赋值）的话被认为是一个隐式的全局变量 _[注: 参考上面的 bar 函数]_ 。 如果一个变量在函数体内的任何位置被赋值，它被认为是一个局部变量，除非明确声明为全局变量。

【下面这段是在没读懂什么意思，防止误导，就先复制过来了。】

Though a bit surprising at first, a moment’s consideration explains this. On one hand, requiring `global` for assigned variables provides a bar against unintended side-effects. On the other hand, if global was required for all global references, you’d be using `global` all the time. You’d have to declare as global every reference to a built-in function or to a component of an imported module. This clutter would defeat the usefulness of the `global` declaration for identifying side-effects.


## 3、为什么定义在循环体中的 lambdas 函数总是会返回一个相同的值?

__原标题: Why do lambdas defined in a loop with different values all return the same result?__

假设使用循环来定义一些不同的 lambda 函数（或者是普通的函数）, eg:

```Python
>>> squares = []
>>> for x in range(5):
...     squares.append(lambda: x**2)
```

你会得到一个包含5个 lambda 函数的 list，函数作用是计算 `x**2`。你可能期望在调用的时候分别返回 `0, 1, 4, 9, 16`。但是，当时真正尝试执行的时候你会发现，它们都会返回 `16`。

```python
>>> squares[2]()
16
>>> squares[4]()
16
```

这是因为变量 `x` 是定义在外部作用域的而并非是 lambda 内部，而且是当 lambda 调用的时候（而不是定义的时候）去访问这个变量的。 循环结束的时候 x 的值是4，所以当前所有的函数都会返回 `4 ** 2` 也就是 `16`。你可以通过改变 x 的值来看看 lambdas 的结果是怎么变化的。

```Python
>>> x = 8
>>> squares[2]()
64
```

为了避免上述情况，你需要在 lambda 函数内部去保存变量的值，这是后就不会依赖于全局变量 x 了

```Python
>>> squares = []
>>> for x in range(5):
...     squares.append(lambda n=x: n**2)
```

这里的 `n = x` 为lambda创建一个新的局部变量n，并在lambda定义时计算，以便它具有与循环中该点处的x相同的值。 这意味着n的值将在第一个lambda中为0，在第二个中为1，第三个中为2，依此类推。 因此，每个lambda现在都会返回正确的结果：

```Python
>>> squares[2]()
4
>>> squares[4]()
16
```

注意这个特性不光是在 lambda 函数中是这样，应用在普通的函数中也是如此。

## 4、如何跨模块共享全局变量？

__原标题: How do I share global variables across modules?__

在单个程序中跨模块共享信息的一个典型的方式是创建一个特殊的模块（通常称为 config 或 cfg ），只需在你应用中的其他模块中导入这个 config 模块，然后模块会把变量变为全局的名称。由于每个模块只有一个实例，因此对模块对象所做的任何更改都会反映到各处。

`config.py`:

```Python
x = 0   # Default value of the 'x' configuration setting
```

`mod.py`:

```Python
import config
config.x = 1
```

`main.py`:

```Python
import config
import mod
print config.x
```

请注意，出于同样的原因，使用模块也是实现 __单例模式(Singleton design pattern)__ 的基础。

## 5、使用 import 命令来引入一个模块的“最佳实践”是怎样的？

__原标题: What are the “best practices” for using import in a module?__

一般来说，不要使用 `from modulename import *` 语句，这样用会使导入者的命名空间混乱，并且使检查未定义的变量名的工作变得更加困难。

在文件顶部导入模块。 这样做可以清楚说明代码需要哪些其他模块，并避免模块名称是否在范围内。 每行使用一个导入可以轻松地添加和删除模块导入，但每行使用多个导入占用的空间更少。

按以下顺序导入模块是一种很好的做法：

+ 1、 标准库，比如 `sys`, `os`, `getopt`, `re`
+ 2、 第三方库（任何安装在 Python 的 site-packages 下的模块），比如 `mx.DateTime`, `ZODB`, `PIL.Image`等等
+ 3、 本地开发的模块

使用明确的相对路径的包来引入，比如你在 `package.sub.m1` 文件写写代码，然后想引入 `package.sub.m2`，不要光写 `import m2`， 即使这样写是合法的。应使用 `from package.sub import m2` 或者 `from . import m2`。

有时为了避免导入循环问题，把 import 引入的内容放在函数体或者类中也是必要的，Gordon McMillan 是这么说的：

  > Circular imports are fine where both modules use the “import <module>” form of import. They fail when the 2nd module wants to grab a name out of the first (“from module import name”) and the import is at the top level. That’s because names in the 1st are not yet available, because the first module is busy importing the 2nd.

  在这种情况下，如果仅在一个函数中使用第二个模块，则可以轻松将导入移动到该函数中。在调用导入时，第一个模块将完成初始化，第二个模块可以导入。

  如果某些模块是特定于平台的，则可能还需要将导入移出顶层代码。在这种情况下，甚至可能无法导入文件顶部的所有模块。在这种情况下，在相应的平台特定的代码中导入正确的模块是一个不错的选择。

  如果需要解决诸如避免循环导入或试图减少模块初始化时间的问题，则只需将导入移动到本地范围内，例如在函数定义内部。根据程序的执行情况，如果许多导入是不必要的，这种技术特别有用。如果这些模块只能用于该功能，您可能还想将导入功能移入功能中。请注意，由于模块的一次初始化，第一次加载模块可能会很昂贵，但多次加载模块几乎是免费的，只需花费几次字典查找。即使模块名称超出范围，该模块可能在 `sys.modules` 中可用。

## 6、Why are default values shared between objects?
## 7、How can I pass optional or keyword parameters from one function to another?
## 8、What is the difference between arguments and parameters?
## 9、Why did changing list ‘y’ also change list ‘x’?
## 10、How do I write a function with output parameters (call by reference)?
## 11、How do you make a higher order function in Python?
## 12、How do I copy an object in Python?
## 13、How can I find the methods or attributes of an object?
## 14、How can my code discover the name of an object?
## 15、What’s up with the comma operator’s precedence?
## 16、Is there an equivalent of C’s “?:” ternary operator?
## 17、Is it possible to write obfuscated one-liners in Python?
## 18、Numbers and strings
