---
title: Programming FAQ - Core Language（语言核心部分）
date: 2018-03-14 19:23:43
tags: [Python, Documentation, Translation]
---

学习编程过程中官方文档真的是非常优秀的学习材料，已经被 **python.org** 深深吸引，以下是 [Programming FAQ（原文链接）](https://docs.python.org/2/faq/programming.html) 目录，本文讨论的是 `Core Language` 部分:

* General Questions
* **Core Language [没错，就是本篇文章]**
* Numbers and strings
* Sequences (Tuples/Lists)
* Dictionaries
* Objects
* Modules

<!-- more -->

## 1、给变量赋值的时候怎么就遇到了 UnboundLocalError 错误呢？

**原标题: Why am I getting an UnboundLocalError when the variable has a value?**

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

这是因为，当对作用域中的变量进行赋值时，该变量将变为 **该作用域的局部变量**，并对在 **外部作用域** 中任何具有 **相同名称的变量** 进行屏蔽。由于 foo 中的最后一条语句 `x += 1` 对 x 进行了 **赋值**，因此解释器将其识别为 **局部变量**。 因此，当前面的 `print x` 尝试打印未初始化的局部变量时产生了错误。

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

**原标题: What are the rules for local and global variables in Python?**

在 Python 中，如果一个变量在函数内仅是被引用（而没有被赋值）的话被认为是一个隐式的全局变量 _[注: 参考上面的 bar 函数]_ 。 如果一个变量在函数体内的任何位置被赋值，它被认为是一个局部变量，除非明确声明为全局变量。

【下面这段是在没读懂什么意思，防止误导，就先复制过来了。】

Though a bit surprising at first, a moment’s consideration explains this. On one hand, requiring `global` for assigned variables provides a bar against unintended side-effects. On the other hand, if global was required for all global references, you’d be using `global` all the time. You’d have to declare as global every reference to a built-in function or to a component of an imported module. This clutter would defeat the usefulness of the `global` declaration for identifying side-effects.

## 3、为什么定义在循环体中的 lambdas 函数总是会返回一个相同的值?

**原标题: Why do lambdas defined in a loop with different values all return the same result?**

假设使用循环来定义一些不同的 lambda 函数（或者是普通的函数）, eg:

```Python
>>> squares = []
>>> for x in range(5):
...     squares.append(lambda: x**2)
```

你会得到一个包含 5 个 lambda 函数的 list，函数作用是计算 `x**2`。你可能期望在调用的时候分别返回 `0, 1, 4, 9, 16`。但是，当时真正尝试执行的时候你会发现，它们都会返回 `16`。

```python
>>> squares[2]()
16
>>> squares[4]()
16
```

这是因为变量 `x` 是定义在外部作用域的而并非是 lambda 内部，而且是当 lambda 调用的时候（而不是定义的时候）去访问这个变量的。 循环结束的时候 x 的值是 4，所以当前所有的函数都会返回 `4 ** 2` 也就是 `16`。你可以通过改变 x 的值来看看 lambdas 的结果是怎么变化的。

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

这里的 `n = x` 为 lambda 创建一个新的局部变量 n，并在 lambda 定义时计算，以便它具有与循环中该点处的 x 相同的值。 这意味着 n 的值将在第一个 lambda 中为 0，在第二个中为 1，第三个中为 2，依此类推。 因此，每个 lambda 现在都会返回正确的结果：

```Python
>>> squares[2]()
4
>>> squares[4]()
16
```

注意这个特性不光是在 lambda 函数中是这样，应用在普通的函数中也是如此。

## 4、如何跨模块共享全局变量？

**原标题: How do I share global variables across modules?**

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

请注意，出于同样的原因，使用模块也是实现 **单例模式(Singleton design pattern)** 的基础。

## 5、使用 import 命令来引入一个模块的“最佳实践”是怎样的？

**原标题: What are the “best practices” for using import in a module?**

一般来说，不要使用 `from modulename import *` 语句，这样用会使导入者的命名空间混乱，并且使检查未定义的变量名的工作变得更加困难。

在文件顶部导入模块。 这样做可以清楚说明代码需要哪些其他模块，并避免模块名称是否在范围内。 每行使用一个导入可以轻松地添加和删除模块导入，但每行使用多个导入占用的空间更少。

按以下顺序导入模块是一种很好的做法：

* 1、 标准库，比如 `sys`, `os`, `getopt`, `re`
* 2、 第三方库（任何安装在 Python 的 site-packages 下的模块），比如 `mx.DateTime`, `ZODB`, `PIL.Image`等等
* 3、 本地开发的模块

使用明确的相对路径的包来引入，比如你在 `package.sub.m1` 文件写写代码，然后想引入 `package.sub.m2`，不要光写 `import m2`， 即使这样写是合法的。应使用 `from package.sub import m2` 或者 `from . import m2`。

有时为了避免导入循环问题，把 import 引入的内容放在函数体或者类中也是必要的，Gordon McMillan 是这么说的：

> Circular imports are fine where both modules use the “import <module>” form of import. They fail when the 2nd module wants to grab a name out of the first (“from module import name”) and the import is at the top level. That’s because names in the 1st are not yet available, because the first module is busy importing the 2nd.

在这种情况下，如果仅在一个函数中使用第二个模块，则可以轻松将导入移动到该函数中。在调用导入时，第一个模块将完成初始化，第二个模块可以导入。

如果某些模块是特定于平台的，则可能还需要将导入移出顶层代码。在这种情况下，甚至可能无法导入文件顶部的所有模块。在这种情况下，在相应的平台特定的代码中导入正确的模块是一个不错的选择。

如果需要解决诸如避免循环导入或试图减少模块初始化时间的问题，则只需将导入移动到本地范围内，例如在函数定义内部。根据程序的执行情况，如果许多导入是不必要的，这种技术特别有用。如果这些模块只能用于该功能，您可能还想将导入功能移入功能中。请注意，由于模块的一次初始化，第一次加载模块可能会很昂贵，但多次加载模块几乎是免费的，只需花费几次字典查找。即使模块名称超出范围，该模块可能在 `sys.modules` 中可用。

## 6、为什么对象之间可以共享默认值?

**原标题: Why are default values shared between objects?**

这种类型的 bug 通常会让新手程序员痛苦，看一下这个函数:

```Python
def foo(mydict={}):  # Danger: shared reference to one dict for all calls
    ... compute something ...
    mydict[key] = value
    return mydict
```

第一次调用这个函数时，mydict 包含了一个单独的 item。 第二次，mydict 包含两个 item，因为当 `foo()` 开始执行时，mydict 从其中已有的 item 开始。

通常期望函数调用为默认值创建新对象。 这不是发生了什么。 当函数被定义时，默认值只创建一次。 如果该对象发生更改（如本示例中的字典），则对该函数的后续调用将引用此已更改的对象。

根据定义，诸如数字，字符串，元组和无的不可变对象可以免于更改。 对可变对象（如字典，列表和类实例）的更改可能会导致混淆。

由于这个特性，不使用可变对象作为默认值是很好的编程习惯。 相反，使用 `None` 作为默认值，并在函数内部检查参数是否为 `None` 并创建一个新的列表/字典/如果是的话。 例如，不要这样写：

```Python
def foo(mydict={}):
    ...
```

而是要这样:

```Python
def foo(mydict=None):
    if mydict is None:
        mydict = {}  # create a new dict for local namespace
```

此功能可能很有用。 当你有一个计算耗时的函数时，一个常用的技术是缓存参数和函数每次调用的结果值，如果再次请求相同的值，则返回缓存的值。 这就是所谓的 `memoizing`(记忆)，可以这样实现:

```Python
# Callers will never provide a third parameter for this function.
def expensive(arg1, arg2, _cache={}):
    if (arg1, arg2) in _cache:
        return _cache[(arg1, arg2)]

    # Calculate the value
    result = ... expensive computation ...
    _cache[(arg1, arg2)] = result           # Store result in the cache
    return result
```

您可以使用包含字典的全局变量而不是默认值，这是一个个人习惯的问题。

## 7、从一个函数到另一个函数如何获取可选参数或者是关键字参数？

**原标题: How can I pass optional or keyword parameters from one function to another?**

在函数参数列表中使用 `*` 和 `**` 符号来收集参数，你会得到一个由 **元组** `(tuple)` 的 **位置参数** `(positional arguments)` 和 **字典** `(dict)` 的 **关键字参数** `(keyword arguments)`，你可以再次使用 `*` 和 `**` 符号将参数传递到另一个函数中：

```Python
def f(x, *args, **kwargs):
    ...
    kwargs['width'] = '14.3c'
    ...
    g(x, *args, **kwargs)
```

Python2.0 之前版本实现的话使用 `apply()` 函数：

```Python
def f(x, *args, **kwargs):
    ...
    kwargs['width'] = '14.3c'
    ...
    apply(g, (x,)+args, kwargs)
```

## 8、arguments 和 parameters 有什么不同？

**原标题: What is the difference between arguments and parameters?**

`parameters` 是通过出现在函数中的名称来定义的，然而 `arguments` 指的是调用函数时具体的参数的值，parameters 定义了函数中什么样 arguments 的类型是可以被接受的。

```Python
def func(foo, bar=None, **kwargs):
    pass
```

foo，bar，kwargs 是函数 `func` 的 parameters，当调用 `func` 时：

```Python
func(42, bar=314, extra=somevar)
```

具体的值 42，314，somevar 就是 arguments。

## 9、改变 list ‘y’ 的值为什么 ‘x’ 的值同时也改变了？

**原标题: Why did changing list ‘y’ also change list ‘x’?**

如果你写如下代码:

```python
>>> x = []
>>> y = x
>>> y.append(10)
>>> y
[10]
>>> x
[10]
```

A specific case is that `a_list.pop([index])` will remove and return item at index (default last).

你可能会感到奇怪：为什么在列表 y 追加了一个元素同时也改变了 x？

两个因素导致了这个结果：

* 1、变量是用来引用某个对象的简单的名字。使用 `y = x` 并不会复制一个列表 - 它是创建了一个叫做 y 的变量并且引用了和变量 x 相同的对象。这意味着仅仅有一个列表对象，并且有 x 和 y 两个变量引用它。
* 2、列表是 [可改变的对象](https://docs.python.org/2/glossary.html#term-mutable)，这意味着你可以改变它们的内容。

调用 `append()` 之后，可变对象的内容从 `[]` 变成了 `[10]`，由于两个变量引用了同一个对象，使用变量会访问已经修改过的 `[10]`。

如果我们把 x 赋值给一不可变对象:

```python
>>> x = 5  # int 值是不可变对象
>>> y = x
>>> x = x + 1  # 5 不能够被改变, 在这里创建了一个新的对象
>>> x
6
>>> y
5
```

通过这个例子我们看到 x 和 y 不再相等了。这是因为整型是 [不可变对象](https://docs.python.org/2/glossary.html#term-immutable)，当执行 `x = x + 1` 的时候不会对 5 的值进行增加，而是创建了一个新的对象（int 6）并且赋值给变量 x，也就是说改变了 x 引用的对象。新赋值之后我们现在有两个对象（int 6 和 5）并且有两个变量引用了他们（x -> 6, y -> 5）。

一些操作（例如 `y.append(10)` 和 `y.sort()`）会对对象进行改变，而表面上类似的操作（例如 `y = y + [10]` 和`sorted(y)` ）会创建一个新对象。通常在 Pythoqn 中（所有示例都是在标准库中），对对象进行改变的方法将返回 `None` 以避免混淆两种操作。所以如果你错误地写 `y.sort()` 认为它会给你一个排序后 y 的复制，你最终会得到 `None`，这可能会导致你的程序产生一个比较容易诊断的错误。

但是，有一类操作，其中相同操作有时具有不同类型的不同行为：`+=` 操作符。例如，`+=` 改变 list，但不改变元组(tuples)或整数(int)，`a_list + = [1,2,3]` 等同于 `a_list.extend([1,2,3])` 并且改变 a_list，而 some_tuple + =（1,2 ，3）和 some_int + = 1 创建新的对象）。

换一种说法：

* 如果我们有一个可变对象（列表，字典，集合等），我们可以使用一些特定的操作来对它进行改变，并且所有引用它的变量都会看到变化。
* 如果我们有一个不可变的对象（str，int，tuple 等），所有引用它的变量将始终看到相同的值，但将该值转换为新值的操作总是返回一个新对象。

如果你想知道两个变量是否指向同一个对象，你可以使用 `is` 运算符或内置函数 `id()`。

## 10、该怎样写一个输出参数变量的函数呢 (通过引用的方式)?

**原标题: How do I write a function with output parameters (call by reference)?**

## 11、在 Python 中如何做一个高阶函数？

**原标题: How do you make a higher order function in Python?**

## 12、在 Python 中如何复制一个对象？

**原标题: How do I copy an object in Python?**

## 13、如何获得一个对象的方法或者属性？

**原标题: How can I find the methods or attributes of an object?**

## 14、我的代码如何获取一个对象的名字？

**原标题: How can my code discover the name of an object?**

## 15、逗号运算符的优先级是怎样的？

**原标题: What’s up with the comma operator’s precedence?**

## 16、有没有与 C 家族语言类似的 “?:” 三元运算符的写法?

**原标题: Is there an equivalent of C’s “?:” ternary operator?**

有的，这个 feature 在 Python 2.5 中被添加进来了，语法如下：

```Python
[on_true] if [expression] else [on_false]

x, y = 50, 25

small = x if x < y else y
```

Python 2.5 版本之前不支持。

## 17、是否有可能使用 Python 编写 one-liners 的复杂代码？

**原标题: Is it possible to write obfuscated one-liners in Python?**
可以的，通常可以用 [lambda](https://docs.python.org/2/reference/expressions.html#lambda) 嵌套 [lambda](https://docs.python.org/2/reference/expressions.html#lambda) 的方式实现，可以看下面三个例子，由 Ulf Bartelt 实现的。

```Python
# 生成质数列表 < 1000
print filter(None,map(lambda y:y*reduce(lambda x,y:x*y!=0,
map(lambda x,y=y:y%x,range(2,int(pow(y,0.5)+1))),1),range(2,1000)))

# 前 10 个斐波那契数列
print map(lambda x,f=lambda x,f:(f(x-1,f)+f(x-2,f)) if x>1 else 1: f(x,f),
range(10))

# 曼德博集合（然而并不知道这是什么东西）
print (lambda Ru,Ro,Iu,Io,IM,Sx,Sy:reduce(lambda x,y:x+y,map(lambda y,
Iu=Iu,Io=Io,Ru=Ru,Ro=Ro,Sy=Sy,L=lambda yc,Iu=Iu,Io=Io,Ru=Ru,Ro=Ro,i=IM,
Sx=Sx,Sy=Sy:reduce(lambda x,y:x+y,map(lambda x,xc=Ru,yc=yc,Ru=Ru,Ro=Ro,
i=i,Sx=Sx,F=lambda xc,yc,x,y,k,f=lambda xc,yc,x,y,k,f:(k<=0)or (x*x+y*y
>=4.0) or 1+f(xc,yc,x*x-y*y+xc,2.0*x*y+yc,k-1,f):f(xc,yc,x,y,k,f):chr(
64+F(Ru+x*(Ro-Ru)/Sx,yc,0,0,i)),range(Sx))):L(Iu+y*(Io-Iu)/Sy),range(Sy
))))(-2.1, 0.7, -1.2, 1.2, 30, 80, 24)
#    \___ ___/  \___ ___/  |   |   |__ lines on screen
#        V          V      |   |______ columns on screen
#        |          |      |__________ maximum of "iterations"
#        |          |_________________ range on y axis
#        |____________________________ range on x axis
```

不要尝试这么写。
