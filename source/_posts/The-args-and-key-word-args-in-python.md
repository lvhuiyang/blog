---
title: 编写可接受任意数量参数的函数
date: 2017-06-07 22:19:20
tags: [Python]
---

编写可接受任意数量参数的函数 (python 函数参数的使用)

<!-- more -->

## 关于位置参数

+ 看一个问题

写一个函数，可以接受任意数量的整形参数，返回这个所有参数的平均值

+ code

```python
def avg(*args):
    return sum(args) / len(args)


if __name__ == '__main__':
    print(avg(1, 2, 3))
    print(avg(11.1, 22.2, 33.3))

```

输出结果：

``` bash
2.0
22.2
```

+ 存在的问题：

``` bash
print(avg())
```

结果：

```python
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    print(avg())
  File "<input>", line 2, in avg
    return sum(args)/len(args)
ZeroDivisionError: division by zero
```

当传参数数量为0时抛出`ZeroDivisionError`异常， 那修改一下问题需求：写一个函数，可以接受任意数量的整形参数（至少是一个），返回这个所有参数的平均值

+ code

```python
def avg(first, *args):
    return (first + sum(args)) / (1 + len(args))


if __name__ == '__main__':
    print(avg(1, 2, 3))
    print(avg(11.1, 22.2, 33.3))
```

输出结果：

``` bash
2.0
22.2
```

这样我们在调用函数的参数数量上做了限制。

## 关于关键字参数

+ 问题

写一个函数，用来构造html内容，具体是接收一个标签的name，一个标签的value，不定数量的key-value对组成的dict数据类型的标签属性。

+ 实例输入输出

调用: `func('a', 'go_to_baidu', **{'href': 'www.baidu.com', 'class': 'button'})`

返回: `<a href='www.baidu.com' class='button'>go_to_baidu</a>`

+ code

```python
def make_html(label_name, label_value, **attr):
    attr_content = ["%s='%s'" % attr_item for attr_item in attr.items()]
    attr_to_str = ' '.join(attr_content)
    return '<{name} {attr}>{value}</{name}>'.format(
        name=label_name,
        attr=attr_to_str,
        value=label_value
    )


if __name__ == '__main__':
    dict_a = {'href': 'www.baidu.com', 'class': 'button'}
    print(make_html('a', 'go_to_baidu', **dict_a))
```

+ 结果

`<a href='www.baidu.com' class='button'>go_to_baidu</a>`

## 出处与小结

__上述实例来自 Python Cookbook 一书，总结引用[知乎路人甲的回答](https://zhuanlan.zhihu.com/p/23526961)__

### 这两个参数是什么意思：\*args，\*\*kwargs？我们为什么要使用它们？

__答__：如果我们不确定往一个函数中传入多少参数，或者我们希望以`元组（tuple`）或者`列表（list）`的形式传参数的时候，我们可以使用\*args（单星号）。如果我们不知道往函数中传递多少个关键词参数或者想传入`字典`的值作为关键词参数的时候我们可以使用\*\*kwargs（双星号），args、kwargs两个标识符是约定俗成的用法。

另一种答法：当函数的参数前面有一个星号\*号的时候表示这是一个可变的位置参数，两个星号\*\*表示这个是一个可变的关键词参数。星号\*把`序列`或者`集合`解包（unpack）成位置参数，两个星号\*\*把`字典`解包成关键词参数。
