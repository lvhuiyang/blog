---
layout: unicode
title: (译)Unicode HOWTO
date: 2018-03-02 19:53:19
tags: Python, Documentation, Translation
---

# Unicode HOWTO

【说明：本文翻译自 Python2.7 官方文档 https://docs.python.org/2/howto/unicode.html 】

主要讨论 Python 2.x’s 对于 Unicode 的支持，并且解释开发者尝试使用 unicode 工作时遇到的各种各样的问题。
对于 Python 3 版本，前往 https://docs.python.org/3/howto/unicode.html

<!-- more -->

## Unicode 的介绍

### 字符编码的历史

1968年，美国信息交换标准代码(American Standard Code for Information Interchange)被标准化，或许它的缩写 ASCII 更广为人知。ASCII 为不同的字符定义了数值型的编码，也就是从 0 到 127的这些数值。比如， 小写字母 ‘a’ 被分配到了 97 作为它的编码值。
ASCII 是美国人发展成熟的一个标准，因此它只是定义了无音节的字符，比如有 ‘e’ 这种定义单数没有 ‘é’ 或者是 ‘Í’。这就意味着有音节的语言就不能够被 ASCII 准确地表达出。（实际上，英语中缺失的音节也会有问题， 比如 ‘naïve’ 和 ‘café’。）

有一段时间，人们编写的是并不能表达音节的程序。我记得看过20世纪80年代中期法语出版的苹果第二代 BASCI 程序（Apple Ⅱ BASIC programs），有这样的语句

```BASCI
PRINT "MISE A JOUR TERMINEE"
PRINT "PARAMETRES ENREGISTRES"
```

这些信息应该是有音节的，并且它们对于那些懂法语的人来看都是错的。

20世纪80年代，几乎所有的计算机的是 8bit的，意味着可以计算的值的范围是 0-255。 ASCII 码占到了 127，所以一些机器就把 128-255 的值分配给了音节字符。不同的机器定义是不同的，导致了文件交换的问题。最终 128-255 范围内常用的集合出现了， 一些被国际标准组织指定为标准， 一些被接二连三的公司指定为标准。

255个字符不是很多，例如，你不能将西欧使用的音节字符和用于俄语的西里尔字母同事放入128-255范围，因为这些字符超过128个。

你可以使用不同的编码写文件（所有的俄语文件都在一个名为KOI8的编码系统中，所有的法语文件都在另一个不同的称为称为Latin1的编码系统中），但是如果你想编写一个引用俄语的法语文件呢？在20世纪80年代，人们开始想要解决这个问题，Unicode标准化工作开始了。

Unicode开始时使用16位字符而不是8位字符。 16位意味着你有2 ^ 16 = 65,536个不同的可用值，可以用许多不同的字母表示许多不同的字符，最初的目标是让Unicode包含每一种人类语言的字母。事实证明，即使16位也不足以实现这一目标，而现代 Unicode 使用了范围更广的 0-1,114,111（基于16进制的0x10ffff）编码。

有一个相关的 ISO 标准，ISO 10646. Unicode 和 ISO 10646最初是各自在发展，但最终范与Unicode的1.1版本合并。

（以上讨论的 Unicode 的历史是简化后的，我并不认为普通的 Python 程序员需要去关心这些历史细节，访问列在参考中的 Unicode 联盟的网站可以得到更多信息。）

### 定义

字符可能是组成文本的最小单元，比如 ‘A’, ‘B’, ‘C’ 或是 ‘È’ 和 ‘Í’等不同的字符。字符是抽象化的，并且非常依赖语言或者是交谈过程中的上下文。例如，欧姆（Ω）的符号通常与希腊字母中的大写字母欧米茄（Ω）非常类似（它们在某些字体中甚至可能是相同的），但这些是具有不同含义的两个不同字符。

Unicode标准描述了如何用 `code points` （代码点）表示字符。 代码点是一个整数值，通常用16进制表示。在标准中，一写作字符 U+12ca 来表示值为 0x12ca（十进制4810）的字符。 Unicode标准包含很多列出字符及其相应代码点，如下：

```
0061    'a'; LATIN SMALL LETTER A
0062    'b'; LATIN SMALL LETTER B
0063    'c'; LATIN SMALL LETTER C
...
007B    '{'; LEFT CURLY BRACKET
```

严格地说，这些定义意味着说 “这是字符 U+12ca” 没有意义。 U+12ca 是一个代码点，代表一些特定的字符; 在这种情况下，它代表字符'ETHIOPIC SYLLABLE WI'。 在非正式情况下，代码点和角色之间的区别有时会被遗忘。

一个字符通过一组称为字形的图形元素显示在屏幕或纸上。 例如，大写字母A的字形是两个对角线笔划和一个水平笔划，但确切的细节将取决于正在使用的字体。 大多数Python代码不需要担心字形; 找出要显示的正确字形通常是GUI工具包或终端的字体渲染器的工作。

### Encodings（编码）

总结上一部分：一个 Unicode 的字符串就是一系列的从数字 0 到 0x10ffff 的代码点。这一序列需要内存中一系列的字节（byte）（意味着 0-255）来表示。将Unicode字符串转换为一系列字节的过程叫做编码 `encoding`。

你可能认为的第一个 encoding 是一个由整形数字组成的 32位的数组。在这个表示方法下，字符串 “Python” 是这样表示的：

```bash
  P           y           t           h           o           n
0x50 00 00 00 79 00 00 00 74 00 00 00 68 00 00 00 6f 00 00 00 6e 00 00 00
  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23
```

这种表示方法很简单,但使用它会带来很多问题。

+ 1、不便捷，不同的处理器的字节顺序是不同的。

+ 2、这太浪费空间。 在大多数文本中，大多数代码点小于127或小于255，因此大量空间被零字节占用。 与ASCII表示所需的6个字节相比，上面的字符串需要24个字节。 增加内存使用量并不重要（台式计算机具有兆字节的内存，并且字符串通常并不那么大），但是将磁盘和网络带宽的使用扩大4倍是不能容忍的。

+ 3、不兼容现有的 C函数，比如 `strlen()`， 因此一套全新的宽字符函数需要被使用。

+ 4、许多互联网标准定义的文本数据,并且不能处理与嵌入式零字节的内容。

通常人们不使用这个编码,而不是选择其他编码更加高效和方便。utf - 8是最广泛支持的编码;这将在下面讨论。

编码不必处理每个可能的Unicode字符，而大多数编码则不需要。 例如，Python的默认编码是'ascii'编码。 将Unicode字符串转换为ASCII编码的规则很简单，对于每一个代码点：

+ 1、如果代码点< 128,每个字节代码点的值是一样的

+ 2、如果代码是128或更高,Unicode字符串不能代表在这个编码。（Python 会在这个示例中抛出 `UnicodeEncodeError` 异常）

Latin-1 编码，也就是 ISO-8859-1 标准是一个与 Unicode 相似的编码。Unicode代码点0 - 255 latin - 1的值是相同的,所以这个编码转换只需要将代码点转换为字节值;如果遇到一个代码点超过255,编码的字符串不能为latin - 1。

编码不一定是简单的一对一映射，如Latin-1。 考虑IBM的EBCDIC，它在IBM大型机上使用。 字母值不在一个块中：'a'到'i'的值为129到137，但'j'到'r'为145到153.如果您想使用EBCDIC作为编码， 使用某种查找表来执行转换，但这主要是内部细节。

UTF-8是最常用的编码之一。 UTF代表“Unicode转换格式”，“8”表示编码中使用8位数字。 （还有一种UTF-16编码，但它的使用频率比UTF-8少。）UTF-8使用以下规则。

+ 1、如果代码点< 128,它是由对应的字节值表示。

+ 2、如果代码是128和0 x7ff之间,就变成了两个字节值在128和255之间。

+ 3、代码点> 0x7ff 会变成三或四字节序列,每个字节的序列是在128和255之间。

UTF-8 有几个方便的特点：

+ 1、它可以处理任何 Unicode 代码点。

+ 2、Unicode字符串转换成一个字符串的字节数不含嵌入式零字节。这避免了字节顺序问题,意味着utf - 8编码的字符串可以处理的C函数如strcpy()和通过协议无法处理零字节。

+ 3、一连串的ASCII文本也是有效的utf - 8的文本。

+ 4、UTF-8相当紧凑; 大部分代码点都变成了两个字节，小于128的值只占用一个字节

+ 5、如果字节是损坏或丢失,可以确定下一个utf - 8编码的代码的开始点和重新同步。,随机的8位数据也不太可能像有效utf - 8。

### 参考资料

【未翻译】

The Unicode Consortium site at <http://www.unicode.org> has character charts, a glossary, and PDF versions of the Unicode specification. Be prepared for some difficult reading. <http://www.unicode.org/history/> is a chronology of the origin and development of Unicode.

To help understand the standard, Jukka Korpela has written an introductory guide to reading the Unicode character tables, available at <https://www.cs.tut.fi/~jkorpela/unicode/guide.html>.

Another good introductory article was written by Joel Spolsky <http://www.joelonsoftware.com/articles/Unicode.html>. If this introduction didn’t make things clear to you, you should try reading this alternate article before continuing.

Wikipedia entries are often helpful; see the entries for “character encoding” <http://en.wikipedia.org/wiki/Character_encoding> and UTF-8 <http://en.wikipedia.org/wiki/UTF-8>, for example.

## Python 2.x 对于 Unicode 的支持

### Unicode 类型

Unicode 字符串是一个 `unicode` 类型的示例， `unicode` 是 Python 内置的保留字。它源于一个抽象类型 `basestring`， `str` 类型也是如此，因此你可以使用  `isinstance(value, basestring)` 检查一个 string类型的变量。在更底层，Python 的 Unicode字符串代表了16或是32位 的整数类型，具体哪个由 Python解释器决定。

`unicode()` 构造函数的签名是 `unicode(string[, encoding, errors])`。 它的参数都应是 8位的字符串类型。第一个参数指明使用哪种编码方式，如果你没有指明 `encoding` 参数， 默认使用 ASCII 编码，因此超过 127的字符被出现如下错误

```Python
>>> unicode('abcdef')
u'abcdef'
>>> s = unicode('abcdef')
>>> type(s)
<type 'unicode'>
>>> unicode('abcdef' + chr(255))
Traceback (most recent call last):
...
UnicodeDecodeError: 'ascii' codec can't decode byte 0xff in position 6:
ordinal not in range(128)
```

`error` 参数指明了当输入的字符串不能够被正确编码时的返回值，此参数变量合法的值是 `strict`（抛出 UnicodeDecodeError 一次）, `replace`（加 U+FFFD， ‘REPLACEMENT CHARACTER’） 或者是 `ignore`（只留下 Unicode的结果），下面的例子说明它们的不同处。

```Python
>>> unicode('\x80abc', errors='strict')     
Traceback (most recent call last):
    ...
UnicodeDecodeError: 'ascii' codec can't decode byte 0x80 in position 0:
ordinal not in range(128)
>>> unicode('\x80abc', errors='replace')
u'\ufffdabc'
>>> unicode('\x80abc', errors='ignore')
u'abc'
```

编码被指定为包含编码名称的字符串。 Python 2.7带有大约100种不同的编码; 请参阅标准编码中的Python库参考以获取列表。 一些编码有多个名字; 例如'latin-1'，'iso_8859_1'和'8859'都是相同编码的同义词。

Python中的内置函数 `unichr()` 可以接收一个整形参数并且返回长度为1的 Unicode 字符串，相对应的是 `ord()` 函数接收一个 Unicode 字符串并且返回对应代码点的值。

```Python
>>> unichr(40960)
u'\ua000'
>>> ord(u'\ua000')
40960
```

`unicode` 类型的示例有许多和 8位字符串类型操作同样的方法，比如查询（searching）和格式化（formatting）：

```Python
>>> s = u'Was ever feather so lightly blown to and fro as this multitude?'
>>> s.count('e')
5
>>> s.find('feather')
9
>>> s.find('bird')
-1
>>> s.replace('feather', 'sand')
u'Was ever sand so lightly blown to and fro as this multitude?'
>>> s.upper()
u'WAS EVER FEATHER SO LIGHTLY BLOWN TO AND FRO AS THIS MULTITUDE?'
```

注意方法的参数可以使 Unicode 字符串或者是8位字符串，8位字符串操作前会同样转换为Unicode；Python 默认使用 ASCII编码，所以超出 127的字符将会抛出异常。

```python
>>> s.find('Was\x9f')
Traceback (most recent call last):
    ...
UnicodeDecodeError: 'ascii' codec can't decode byte 0x9f in position 3:
ordinal not in range(128)
>>> s.find(u'Was\x9f')
-1
```

上运行的Python代码字符串将因此与Unicode字符串不需要任何更改代码。(输入和输出为Unicode代码需要更多的更新;后面详细讨论)。

另一个重要的方法是 `.encode([encoding], [errors='strict'])`，此方法被编码为某一编码格式并且返回8位的Unicode 字符串。`errors` 参数和 `unicode()` 的构造函数除了‘strict’, ‘ignore’, and ‘replace’相同外还有个额外参数 `xmlcharrefreplace`，代表使用 XML的字符引用。以下例子展示不同的结果：

```python
>>> u = unichr(40960) + u'abcd' + unichr(1972)
>>> u.encode('utf-8')
'\xea\x80\x80abcd\xde\xb4'
>>> u.encode('ascii')                       
Traceback (most recent call last):
    ...
UnicodeEncodeError: 'ascii' codec can't encode character u'\ua000' in
position 0: ordinal not in range(128)
>>> u.encode('ascii', 'ignore')
'abcd'
>>> u.encode('ascii', 'replace')
'?abcd?'
>>> u.encode('ascii', 'xmlcharrefreplace')
'&#40960;abcd&#1972;'
```

Python 的8位字符串有一个 `.decode([encoding], [errors])` 方法皆可使用指定的编码进行解码。

```python
>>> u = unichr(40960) + u'abcd' + unichr(1972)   # Assemble a string
>>> utf8_version = u.encode('utf-8')             # Encode as UTF-8
>>> type(utf8_version), utf8_version
(<type 'str'>, '\xea\x80\x80abcd\xde\xb4')
>>> u2 = utf8_version.decode('utf-8')            # Decode using UTF-8
>>> u == u2                                      # The two strings match
True
```

编解码器模块中提供了用于注册和访问可用编码的低级例程[codecs](https://docs.python.org/2/library/codecs.html#module-codecs)。 但是，这个模块返回的编码和解码函数通常比舒适的更低级，所以我不打算在这里描述编解码器模块。 如果您需要实现全新的编码，您需要了解编解码器模块接口，但实现编码是一项特殊任务，这里也不会涉及。 请参阅Python文档以了解关于此模块的更多信息。

 [codecs](https://docs.python.org/2/library/codecs.html#module-codecs)模块中最常用的部分是 [codecs.open()](https://docs.python.org/2/library/codecs.html#codecs.open) 函数，关于这个函数会在输入输出章节的部分进行讨论。

### Unicode Literals in Python Source Code

### Unicode Properties

### References

## Reading and Writing Unicode Data

### Unicode filenames

### Tips for Writing Unicode-aware Programs

### 参考

【未翻译】

The PDF slides for Marc-André Lemburg’s presentation “Writing Unicode-aware Applications in Python” are available at <https://downloads.egenix.com/python/LSM2005-Developing-Unicode-aware-applications-in-Python.pdf> and discuss questions of character encodings as well as how to internationalize and localize an application.

### 版本的修订和确认

【未翻译】

Thanks to the following people who have noted errors or offered suggestions on this article: Nicholas Bastin, Marius Gedminas, Kent Johnson, Ken Krugler, Marc-André Lemburg, Martin von Löwis, Chad Whitacre.

Version 1.0: posted August 5 2005.

Version 1.01: posted August 7 2005. Corrects factual and markup errors; adds several links.

Version 1.02: posted August 16 2005. Corrects factual errors.

Version 1.03: posted June 20 2010. Notes that Python 3.x is not covered, and that the HOWTO only covers 2.x.
