---
title: Python 元编程
date: 2017-10-28 20:41:41
tags: 
    - Python
    - 《Python Cookbook》
---

《Python Cookbook》 Chapter 9. Metaprogramming 学习笔记

软件开发中最重要的一条真理就是 “不要重复自己的工作（Don't repeat yourself）”。也就是说，任何时候当你的程序中存在高度重复(或者是通过剪切复制) 的代码时，都应该想想是否有更好的解决方案。在Python 当中，通常都可以通过元编程来解决这类问题。简而言之，元编程就是关于创建操作源代码(比如修改、生成或包装原来的代码) 的函数和类。主要技术是使用装饰器、类装饰器和元类。

<!-- more -->

## Putting a Wrapper Around a Function

### Problem

You want to put a wrapper layer around a function that adds extra processing (e.g., logging, timing, etc.).

### Solution

If you ever need to wrap a function with extra code, define a decorator function. For example:

```python
import time
from functools import wraps

def timethis(func):
    '''
    Decorator that reports the execution time.
    '''
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(func.__name__, end-start)
        return result
    return wrapper
```

Here is an example of using the decorator:

```python
>>> @timethis
... def countdown(n):
...     '''
...     Counts down
...     '''
...     while n > 0:
...             n -= 1
...
>>> countdown(100000)
countdown 0.008917808532714844
>>> countdown(10000000)
countdown 0.87188299392912
>>>
```