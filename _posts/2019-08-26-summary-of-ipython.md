---
layout: post
title: 关于 IPython 使用的个人小结
categories: python
description: 关于 IPython 使用的个人小结
tags: IPython
---

IPython 是 Python shell 的增强版，它鼓励一种「执行-探索」式(execute explore) 的工作模式，而不是其他许多语言那种「编辑-编译-运行」(edit-compile-run) 的传统工作模式。而且，它与操作系统 shell 和文件系统之间也有非常紧密的集成，因此，在数据分析工作中得到广泛应用。

IPython 官网为 <https://ipython.org/>.

## 1 基础操作

IPython 可通过 pip 进行直接安装 `pip install ipython`，使用命令行启动 IPython 与启动标准 Python 解释器类似：

```bash
$ ipython
Python 3.7.4 (default, Jul  9 2019, 18:13:23)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.7.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from datetime import d<Tab>
                               date          dbm
                               datetime      dim
                               datetime_CAPI
```

相较于 Python 标准解释器，IPython 提供的一些额外功能包括：

- 使用 <kbd>Tab</kbd> 实现自动补全功能。
- 使用问号 <kbd>?</kbd> 呈现对象的说明。
- 提供了一系列以 <kbd>%</kbd> 开头的魔术命令为常见任务的执行提供便利。
- 提供了额外的绘图功能。
- 可使用操作系统中的命令。

在 IPython 环境中，我们可以直接使用 `%quickref` 查看 IPython 的参考手册，也可以使用 `%magic` 来查看 IPython 中魔术命令的信息，以更好的使用 IPython 工具。

```python
In [1]: %quickref
IPython -- An enhanced Interactive Python - Quick Reference Card
================================================================

obj?, obj??      : Get help, or more help for object (also works as
                   ?obj, ??obj).
?foo.*abc*       : List names in 'foo' containing 'abc' in them.
%magic           : Information about IPython's 'magic' % functions.

Magic functions are prefixed by % or %%, and typically take their arguments
without parentheses, quotes or even commas for convenience.  Line magics take a
single % and cell magics are prefixed with two %%.

Example magic function calls:
:
```

**Tips:** IPython 可进行一些[自定义的配置](https://ipython.readthedocs.io/en/stable/config/index.html)，该配置文件位于 `~/.ipython` 或 `~/.config/.ipython` 目录（Unix）和 `%HOME%/.ipython` 目录（Windows）中。

## 2 命令历史

IPython 与 Python 解释器一样，采用「逐行输入代码」并「立即执行」的方式，它们均可用<kbd>Ctrl</kbd> + <kbd>P</kbd> 或 <kbd>↑</kbd> 查找执行过的命令、使用 <kbd>Ctrl</kbd> + <kbd>N</kbd> 或 <kbd>↓</kbd>查找（当前查找位置）命令后执行的命令。

此外，IPython 提供了更强的输入输出变量管理以及记录功能：

最近两次执行的结果 (Out) 分别保存在 `_` 及 `__` 变量中。而且，IPython 还采用 `_iN`, `_N` 的方式将所有输入及输出的历史记录进行保存 ~ 其中 N 为行号：

```python
In [1]: 2 + 4 / 1.2
Out[1]: 5.333333333333334

In [2]: _
Out[2]: 5.333333333333334

In [3]: 3 ** 5
Out[3]: 243

In [4]: 1 + 7*9
Out[4]: 64

In [5]: __
Out[5]: 243

In [6]: _i3
Out[6]: '3 ** 5'

In [7]: _3
Out[7]: 243
```

同时，因在 `_iN` 中保存的输入历史为字符串，使用 `eval(_iN)` 即可重新执行指定行中的代码：

```python
In [8]: eval(_i4) # eval('1 + 7*9')
Out[8]: 64
```

对于在 IPython 中执行过的命令，可通过 `%logstart` 将之保存在 Python 文件中：

```python
In [9]: %logstart
Activating auto-logging. Current session state plus future input saved.
Filename       : ipython_log.py
Mode           : rotate
Output logging : False
Raw input log  : False
Timestamping   : False
State          : active
```

该魔术命令会将截止目前输入的命令保存在当前文件夹下的 `ipython_log.py` 文件中，并将后续执行的命令也会添加进去。当前 `ipython_log.py` 文件中的内容为：

```python
# IPython log file

2 + 4 / 1.2
_
3 ** 5
1 + 7*9
__
_i3
_3
eval(_i4)
get_ipython().run_line_magic('logstart', '')
```

## 3 与操作系统交互

IPython 中可执行系统中的命令，以 <kbd>!</kbd> 开头的命令行后的内容需要在系统 shell 中执行，并且可将执行的结果存放在变量中：

```python
In [22]: !ls
IPython.md                      data_science.md                 pandas.ipynb
Jupyter.ipynb                   README.md

In [23]: markdown_file = '*.md'

In [24]: !ls $markdown_file
IPython.md      README.md       data_science.md

In [25]: markdown = !ls $markdown_file

In [26]: markdown
Out[26]: ['IPython.md', 'README.md', 'data_science.md']
```

如上例所示，IPython 中执行 `!` 开始的系统命令时，还可以执行使用当前环境中的变量 ~ 以 `$` 开头。

**Tips:** 我们同样可以使用「魔术命令」 `%alias` 为 shell 命令自定义简称 ~ 仅对当前 IPython 会话有效。要永久有效，可在配置文件中进行设定；也可使用 `%bookmark` 命令来设定常用目录的简写。

## 4 分析工具

IPython 同样提供了一些代码分析工具，如代码执行时间的分析工具 `%time` 和 `%timeit`、基本的性能分析 `%prun` 和 `%run -p` 等。

在 Python 代码中，我们可以借助于 `time` 模块来测算代码运行的时间：

```python
import time

start = time.time()

# do somthing

duration = time.time() - start
```

而 IPython 中提供了 `%time` 和 `%timeit` 两个魔术命令用于简化测算代码运行时间工作：

- `%time`：整体执行一次，给出执行时间；
- `%timeit`：多次执行，给出平均时间。

```python
In [42]: %time 'foobar'.startswith('foo')
CPU times: user 3 µs, sys: 0 ns, total: 3 µs
Wall time: 5.96 µs
Out[42]: True

In [43]: %time 'foobar'[:3] == 'foo'
CPU times: user 2 µs, sys: 1 µs, total: 3 µs
Wall time: 5.01 µs
Out[43]: True

In [44]: %timeit 'foobar'.startswith('foo')
137 ns ± 0.29 ns per loop (mean ± std. dev. of 7 runs, 10000000 loops each)

In [45]: %timeit 'foobar'[:3] == 'foo'
119 ns ± 0.33 ns per loop (mean ± std. dev. of 7 runs, 10000000 loops each)
```

使用 `%time` 运行一次给出运行时间可能会导致一定的偏差；`%timeit` 以多次运行给出平均时间方式更加精准。

此外，`%run -p` 命令为 Python 的性能分析工具 cProfile 模块提供了一个 IPython 的接口，在执行一个程序或代码块时，可记录各函数所耗费的时间。

而 `%debug` 命令则可调用 IPython 中增强版的 pdb 对上一句执行的命令进行调试。

## 5 参考资料

1. McKinney, Wes. [Python for data analysis: Data wrangling with Pandas, NumPy, and IPython](https://github.com/wesm/pydata-book). " O'Reilly Media, Inc.", 2012.
2. IPython Docs, <https://ipython.readthedocs.io/en/stable/index.html>, 2019/08/25.
