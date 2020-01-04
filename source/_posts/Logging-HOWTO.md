---
title: Logging HOWTO
date: 2018-05-12 14:29:50
tags: [Python, Documentation, Translation]
---

【翻译自 Python2.7 官方文档 [https://docs.python.org/2/howto/logging.html](https://docs.python.org/2/howto/logging.html) 】

[Python HOWTOs](https://docs.python.org/2/howto/index.html) 是覆盖单个特定主题的文档，并尝试完全覆盖 Python 文档。 以 Linux 档项目的 HOWTO 为模型，尝试打造比 Python 库参考更详细的文档。

当前包括：

* Porting Python 2 Code to Python 3
* Porting Extension Modules to Python 3
* Curses Programming with Python
* Descriptor HowTo Guide
* Idioms and Anti-Idioms in Python
* Functional Programming HOWTO
* **Logging HOWTO [本篇文章]-[✓]**
* Logging Cookbook
* Regular Expression HOWTO
* Socket Programming HOWTO
* Sorting HOW TO
* [Unicode HOWTO](/2018/03/02/Unicode-HOWTO/) [✓]
* HOWTO Fetch Internet Resources Using urllib2
* HOWTO Use Python in the web
* Argparse Tutorial

<!-- more -->

> Author: Vinay Sajip `<`vinay_sajip at red-dove dot com`>`

## Logging 基本教程

`Logging` 是跟踪某些软件运行时发生的事件的一种方式。 软件的开发人员将日志记录调用添加到他们的代码中，以示发生了某些事件。 事件由描述性消息描述，其可以可选地包含可变数据（即对于事件的每次发生可能不同的数据）。 事件也具有开发者归因于事件的重要性; 重要性也可以称为 _level_ 或 _severity_。

### 什么时机使用 logging

`Logging` 为简单的日志记录使用提供了一组便利的方法(function)，分别是是 `debug()`，`info()`，`warning()`，`error()` 和 `critical()`。 要确定何时使用 logging，请参见下面的表格，该表格针对一组常见任务说明了使用日志的最佳工具。

|                             要执行的任务                             | 最佳工具                                                                                                                                                                                                                                                                                               |
| :------------------------------------------------------------------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|             对于普通的命令行脚本或者程序展示控制台的输出             | [print( )](https://docs.python.org/2/library/functions.html#print)                                                                                                                                                                                                                                     |
|         记录程序中正常操作的事件（比如是状态监测和故障调查）         | [logging.info( )](https://docs.python.org/2/library/logging.html#logging.info)，如果是用于诊断目的要非常详尽的输出的话使用 [logging.debug( )](https://docs.python.org/2/library/logging.html#logging.debug)                                                                                            |
|                   关于一个特定的运行时事件发出警告                   | 如果问题是可以避免,客户端应用应该被修改以消除警告，使用 [warnings.warn( )](https://docs.python.org/2/library/warnings.html#warnings.warn) ; 如果客户端应用程序无法处理这种情况，那么该时间应当被关注，使用 [logging.warn( )](https://docs.python.org/2/library/logging.html#logging.warning) 。       |
|                  对于一个特定的运行时的事件报告错误                  | 抛出一个异常                                                                                                                                                                                                                                                                                           |
| 抑制错误而不引发异常（例如，长时间运行的服务器进程中的错误处理程序） | 根据特定的错误和应用程序域选择 [logging.error( )](https://docs.python.org/2/library/logging.html#logging.error)， [logging.exception( )](https://docs.python.org/2/library/logging.html#logging.exception) 或者 [logging.critical( )](https://docs.python.org/2/library/logging.html#logging.critical) |

logging 函数是根据它们用于追踪的事件的级别或严重程度命名的。 标准级别及其适用性如下所述（按严重性的升序排列）：

| 等级       | 使用时机                                                                            |
| :--------- | :---------------------------------------------------------------------------------- |
| `DEBUG`    | 详细信息，通常只有在诊断问题时才有意义。                                            |
| `INFO`     | 确认事情按预期工作。                                                                |
| `WARNING`  | 出现意外事件或在不久的将来出现某些问题（例如“磁盘空间不足”）。 该软件仍按预期工作。 |
| `ERROR`    | 由于一系列严重的问题，该软件不能执行某些功能。                                      |
| `CRITICAL` | 严重错误，表示程序本身可能无法继续运行。                                            |

### 一个简单的例子

一个简单的例子：

```Python
import logging
logging.warning('Watch out!')  # 会在控制台打印消息
logging.info('I told you so')  # 不回打印任何东西
```

如果把上述代码写入脚本文件中并执行，你将会看到：

```Bash
WARNING:root:Watch out!
```

打印在控制台。`INFO` 消息不会显示，因为默认级别是 `WARNING`。 打印的消息包括对日志调用中提供的事件的级别和描述，即 'Watch out!'。 不要担心 'root' 部分：稍后会解释。 如果你需要的话，实际的输出格式可以非常灵活; 格式化选项也将在稍后解释。

### Logging 记录在文件中

一种非常常见的情况是在文件中记录日志记录事件，下面我们来看看：

```python
import logging
logging.basicConfig(filename='example.log',level=logging.DEBUG)
logging.debug('This message should go to the log file')
logging.info('So should this')
logging.warning('And this, too')
```

接下来我们打开对应生成的文件，我们能够看到日志信息：

```bash
DEBUG:root:This message should go to the log file
INFO:root:So should this
WARNING:root:And this, too
```

此示例还显示了如何设置充当跟踪阈值的日志记录级别。在这个示例中，我们设置阈值为 `DEBUG`，所以我们看到了所有打印的信息。

如果你想通过命令行操作参数来设置日志的级别，如下：

```bash
--log=INFO
```

你能够得到通过 `--log` 获得的 可变的日志级别 `loglevel`，使用：

```python
getattr(logging, loglevel.upper())
```

通过 [basicConfig( )](https://docs.python.org/2/library/logging.html#logging.basicConfig) 获得 _level_ 参数的值。

```python
# assuming loglevel is bound to the string value obtained from the
# command line argument. Convert to upper case to allow the user to
# specify --log=DEBUG or --log=debug
numeric_level = getattr(logging, loglevel.upper(), None)
if not isinstance(numeric_level, int):
    raise ValueError('Invalid log level: %s' % loglevel)
logging.basicConfig(level=numeric_level, ...)
```

```python
logging.basicConfig(filename='example.log', filemode='w', level=logging.DEBUG)
```

### Logging from multiple modules

### Logging variable data

### Changing the format of displayed messages

### Displaying the date/time in messages

### Next Steps

## Advanced Logging Tutorial

### Logging Flow

### Loggers

### Handlers

### Formatters

### Configuring Logging

### What happens if no configuration is provided

### Configuring Logging for a Library

## Logging Levels

### Custom Levels

## Useful Handlers

## Exceptions raised during logging

## Using arbitrary objects as messages

## Optimization
