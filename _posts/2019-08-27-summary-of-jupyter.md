---
layout: post
title: 关于 Jupyter 的使用说明
categories: python
description: 关于 Jupyter 的使用说明 
tags: jupyter
---

Jupyter 是一项基于 IPython 的开源工具，它支持各种编程语言，因其在交互式工作中的良好表现而被广泛应用于「数据科学」及「科学计算」领域。

其官网为：<https://jupyter.org>.

相较于 IPython 而言，二者最大的区别在于 IPython 是基于本地的命令行工具，而 Jupyter 是则基于网页。因此，Jupyter 除支持 IPython 的功能外，它采用基于 JSON 的文档格式 `.ipynb` 进行保存（更便于保存及分享），引入 [编辑模式](#2-%e4%b8%80%e4%ba%9b%e9%87%8d%e8%a6%81%e7%9a%84%e6%a6%82%e5%bf%b5
) 支持 Markdown 格式，且可以在云服务器中运行。

当然，要充分利用 Jupyter 还是离不开基础的 IPython 功能，可参见我 [之前的博客](https://mirrortan.com/2019/08/26/summary-of-ipython/) 以及 [IPython 官方文档](https://ipython.org/) 进行学习。

## 1 安装与运行

官网推荐可采用以下 `conda` 或 `pip` 两种方式进行安装：

```bash
conda install -c conda-forge jupyter
```

or

```bash
pip install jupyter
```

**注意：**建议使用国内镜像源，否则速度可能很慢：

```bash
$ cat > ~/.pip/pip.conf
[global]
index-url=http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
```

安装完成后，可直接使用 `jupyter notebook` 命令运行，该命令会自动打开默认浏览器并转到  <http://127.0.0.1:8888> （若未自动打开，可在浏览器输入该 URL）。

打开 <http://127.0.0.1:8888> 后，会进入 Notebook 的仪表板，会呈现当前目录下的各种文件，点击右上角的「New -> Python3」即可新建一个基于 Python3 的 Jupyter Notebook 文件。

当进入 Notebook 文件后，**强烈建议**点击 「菜单栏」 中「Help -> User Interface Tour」快速了解一些基础操作。

## 2 一些重要的概念

Cell, Jupyter 通过「单元格」来进行组织。每个单元格均可以为「标记」或「代码」中的一种：

- 标记：即可使用 Markdown 格式的文本，主要用于进行说明；
- 代码：即进行代码撰写，使用方式与 IPython 中的输入类似。

此外，每个单元格（包括「标记单元格」及「代码单元格」）均有「命令模式」及「编辑模式」两种模式：

- 命令模式：选中单元格后按 <kbd>Esc</kbd> 即进入命令模式，可输入各种快捷命令，实现对单元格的「添加」、「删除」以及「运行代码」等一系列操作。
- 编辑模式：选中单元格后按 <kbd>Enter</kbd> 即可进入编辑模式，对单元格中的内容进行编辑，输入代码或 Markdown 格式的文本。

**Tips:** 当单元格为「命令模式」时，单元格中的外框线为蓝色；当单元格进入「编辑模式」时，单元格外边框线为绿色。

## 3 快捷键

Jupyter 中的一些快捷键可加快常用的操作，所有可用的快捷键可通过「Help->Keyboard Shortcuts」查看，以下是个人使用中常用的一些快捷键：

1. 查找帮助类：
  - <kbd>H</kbd>：可调出所有可用的快捷键。
  - <kbd>F</kbd>：在当前 Notebook 中进行字符查找及替换。
  - <kbd>P</kbd>：搜索可用的快捷键。
2. 单元格操作类：
  - <kbd>A</kbd>：在当前单元格上方插入单元格。
  - <kbd>B</kbd>：在当前单元格下方插入单元格。
  - <kbd>Ctrl</kbd> + <kbd>Enter</kbd>：运行当前选中单元格。
  - <kbd>Shift</kbd> + <kbd>Enter</kbd>：运行当前单元格并选择或插入新的单元格。
  - <kbd>D</kbd>, <kbd>D</kbd>：删除选中单元格。
  - <kbd>J</kbd>：选中上一个单元格。
  - <kbd>K</kbd>：选中下一个单元格。
  - <kbd>L</kbd>：显示或关闭当前单元格行号。
3. 类型模式转换类：
  - <kbd>Y</kbd>：单元格由「标记」格式转化为「代码」格式。
  - <kbd>M</kbd>：单元格由「代码」格式转化为「标记」格式。
  - <kbd>Esc</kbd>：由「编辑模式」转化为「命令模式」。
  - <kbd>Enter</kbd>：由「命令模式」转化为「编辑模式」。

**注意：**上述快捷键除 <kbd>Esc</kbd> 是在「编辑模式」中使用的外，其余快捷键均需在「命令模式」中使用。

## 4 配置及插件

### 4.1 自定义配置

Jupyter 继承自 IPython，也可以进行自定义的设置，相关配置文件默认位于 `~/.jupyter` 目录，具体配置文件可通过以下命令查看：

```bash
# 查看配置文件的位置
$ jupyter --config-dir
~/.jupyter
# 查看数据文件的位置
$ jupyter --data-dir
~/Library/Jupyter
```

当然，也可以通过设置环境变量 `JUPYTER_CONFIG_DIR` 来改变配置文件的位置。

Python 配置文件可通过以下命令创建：

```bash
jupyter notebook --generate-config
```

该命令会在 `~/.jupyter` 目录下生成 `jupyter_notebook_config.py` 文件，通过编辑该文件可设定一些配置，如在配置文件中写入 `c.NotebookApp.port = 8754`, 则可改变打开 Notebook 时的端口。

### 4.2 插件拓展

除可通过设定配置文件的方式来自定义 Jupyter 外，也可以利用 Jupyter 的拓展来丰富其功能，安装插件的命令为：

```bash
jupyter nbextension install [packages]
```

除此之外，我们也可以利用第三方的插件 [jupyter_contrib_nbextensions](https://jupyter-contrib-nbextensions.readthedocs.io/en/latest/) 实现对插件的管理：

```bash
pip install jupyter_contrib_nbextensions && jupyter contrib nbextension install
```

当安装完该插件管理器后，重新打开 Jupyter Notebook 时首页会多出一个 Nbextensions 的选项卡，可通过可视化的方式完成 Jupyter 的插件安装。

其中个人比较常用的插件有：

- Table of Contents：导航栏
- Codefolding：「代码」折叠
- Autopep8：「代码」格式调整，符合 pep8 规范

这些插件实际上就是 JavaScript 和 CSS 配置文件，用以丰富 Jupyter Notebook 的功能。

## 5 个人使用的技巧

多用于探索式的数据分析，通常我们需要加入 `%matplotlib inline` 让我们画的图内嵌于 Notebook 中。

## 6 其他相关说明

### 6.1 文件存储

我们可以将撰写的 Jupyter 文档（.ipynb) 格式文档另存为 `py, md, pdf, html` 等多种格式，只需在打开的 Notebook 中通过「File -> Download as」并选择相应的格式即可。

将 Notebook 转化为其他格式的文件是利用 nbconvert 来实现的，另存为 PDF 格式的文件需要额外 [安装 pandoc 库](https://pandoc.org/installing.html)。

**Tips:** 将 Notebook 下载为 Reveal.js slides(.slides.html) 格式，可将 Notebook 转化为便于展示的格式。

### 6.2 权限管理与共享

包含整个计算过程及结果的 Jupyter Notebook 文档即可以通过常见的「邮件」、「版本控制系统」(git/github)」进行分享之外，也可以将运算结果通过 [nbviewer.ipython.org](https://nbviewer.jupyter.org/) 在 web 上进行展示和分享。

而要进行权限管理及多人共享的 Jupyter Notebook 环境，需要借助于 [Jupyterhub](https://github.com/jupyterhub/jupyterhub) 实现。

### 6.3 并行计算

Jupyter 还支持并行计算，该功能通过由 [IPython parallel](https://github.com/ipython/ipyparallel) 提供。

```bash
$ pip install ipyparallel
# 在 Jupyter Notebook 中启用
$ ipcluster nbextension enable
```

安装完成后，可通过 Notebook 首页的 [Dashboard](http://localhost:8888/tree#clusters) 中进行管理。当然，也可以通过 [jupyter_notebook_config.py](https://jupyter-notebook.readthedocs.io/en/latest/public_server.html) 配置文件进行管理。

## 7 更多关于 Jupyter 的使用

Jupyter Family 中工具较多，各种工具的图示关系如下：

![Jupyter Family Toolkits relationships](https://jupyter.readthedocs.io/en/latest/_images/repos_map.png)

更多相关操作详见 [官方文档](https://jupyter.org/documentation)。

## 8 参考资料

1. Jupyter Notebook 官方文档, <https://jupyter-notebook.readthedocs.io/en/stable/>, 2019/08/27.
2. Jupyter contrib nbextensions 官方文档, <https://jupyter-contrib-nbextensions.readthedocs.io/en/latest/>, 2019/08/27.
3. Jupyterhub 官方文档, <https://jupyterhub.readthedocs.io/en/stable/>, 2019/08/27.
4. McKinney, Wes. Python for data analysis: Data wrangling with Pandas, NumPy, and IPython. " O'Reilly Media, Inc.", 2012.
