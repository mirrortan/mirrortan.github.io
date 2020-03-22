---
layout: post
title: Python 版本及虚拟环境管理
categories: Python
description: Python 版本及虚拟环境管理
tags: [pipenv, pyenv]
---

实际上，Python 环境的管理可分为两部分：

- 系统 Python 版本的管理
- 工作环境 Project 中的 Python 环境的管理

因 Python 大小版本众多，同时在一个工程项目中使用到的第三方库也较多，因此有各种 Python 环境管理的工作。

## 1 Python 版本管理

可通过 [pyenv][1] 来实现 Python 版本的管理 ~ pyenv 并不是管理虚拟环境的工具，但可配合 `pipenv` 等工具一同使用。

pyenv 的工作原理是通过在 `shims` 目录中创建 Python 解释器的伪造应用，当系统寻找 python 时，它会先在 `shims` 目录中找对应的假版本，传递命令到 pyenv 中。pyenv 再根据环境变量、`.python-version` 文件及全局默认设置信息来判断该运行的版本。

Mac 电脑可通过 `brew install pyenv` 安装 pyenv，使用 pyenv 安装 Python：

```python
pyenv install 3.6.10
# 国内可指定安装的源以加快速度
mkdir -p ~/.pyenv/cache
brew install wget         # 如无 wget 可使用 brew 安装
v=3.7.3;wget https://npm.taobao.org/mirrors/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/;pyenv install $v
$ pyenv versions          # 查看当前有的 Python 版本
* system (set by /Users/***/.pyenv/version)
  3.6.10
```

安装是需指定版本号，关于 `.pyenv/cache` 的设置可参考 [Prasanta][2] 的博客。

## 2 Python 包管理

Python 包安装本质上就是从 [Python Package Index][3] 中下载并安装第三方库，最基础的工具如 `pip, Easy Install`，也有更进一步的工具如 `virtualenv, venv, pipenv` 等。

### 2.1 设置国内源

因各种原因，使用默认的 `pypi` 安装第三方库时比较缓慢，可通过设置国内镜像源来解决这一问题。

通过编辑 `~/.pip/pip.conf` 文件进行设置：

```bash
[global]
index-url=http://mirrors.aliyun.com/pypi/simple/
trusted-host=mirrors.aliyun.com
```

一次性使用的源 `pip install -i http://mirrors.aliyun.com/pypi/simple/ flask`.

此外，可以在 `~/.bashrc` 中设置 `export PIP_REQUIRE_VIRTUALENV=true` 强制要求 `pip install` 将包安装在激活的虚拟环境中，或在 `~/.pip/pip.conf` 中添加 `require-virtualenv = true`。

### 2.2 Pipenv

Python 中的第三方库管理工具中，[`Pipenv`][4] 使用体验好，可替代 `pip` 和 `virtualenv` 两个工具。

#### 2.2.1 Install

在 Mac 中可通过 `brew` 安装 `pipenv`：

```bash
brew install pipenv
```

#### 2.2.2 Usage

使用 `pipenv` 进行工作的一般流程为：

```bash
cd [PROJECT PATH]
pipenv --three      # 基于 Python 3 创建虚拟环境
pipenv --python 3.6 # 指定 Python 版本创建虚拟环境
# do something
exit                # 退出当前环境
pipenv shell        # 进入工作环境
# do something
exit                # 退出当前环境
pipenv --rm         # 删除当前虚拟环境
```

`Pipenv` 会默认创建 `Pipfile` 和 `Pipfile.lock`，`Pipfile` 包含所有已安装软件包的列表 ~ 并且可以进行不同开发环境的管理（便于人阅读）；`Pipfile.lock` 中包含当前环境维护的具体内容。

同时，在 `pyenv` 已安装的情况下，可自动安装对应 Python 版本。

此外，当使用 Git 进行管理时，将 `Pipfile, Pipfile.lock` 纳入版本管理，当克隆仓库后，只要使用 `pipenv install` 即可完成环境的配置。

#### 2.2.3 Setting

默认情况下，使用 `pipenv` 管理的虚拟环境位于用户的主目录下，可通过在 `.bashrc/.zshrc` 中设置 `export PIPENV_VENV_IN_PROJECT=1` 让生成的虚拟环境位于工程主目录内。

在工程环境中，可在主目录中创建 `.env` 文件，当进入虚拟环境中时，会将 `.env` 文件中的变量载入工作环境中。

## 3 小结

因此，综合而言，推荐使用 `pyenv` 来进行 Python 版本管理，使用 `pipenv` 来进行工程虚拟环境及包管理。

## 4 参考资料

1. [Pyenv docs][1], 2020/3/15.
2. [Pipenv 官网][4], 2020/3/15.
3. [pyenv 安装配置与国内镜像加速 结合 virtualenv][2], [Prasanta](https://segmentfault.com/blog/xiuxian), 2020/3/15.

[1]: https://github.com/pyenv/pyenv "pyenv"
[2]: https://segmentfault.com/a/1190000006174123 "pyenv 安装配置与国内镜像加速 结合 virtualenv"
[3]: https://pypi.org/ "pypi"
[4]: https://pipenv.readthedocs.io/en/latest/ "Pipenv 官网"
