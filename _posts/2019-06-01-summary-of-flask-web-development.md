---
layout: post
title: 关于 Flask Web 开发的个人总结
categories: Python
tags: [Flask, Web-Development]
---

在阅读完 Miguel Grinberg 的 *Flask Web Development: Developing Web Application with Python* 并按照书中的案例进行了一遍实操，对 Miguel Grinberg 的 **flasky** 以及 Grey Li 的 **bluelog** 简单改写后做了一个[自己的 Flask 网站](https://www.mirrortan.com)，现对 Flask Web 做一个简单的个人小结。

## 1 简介

Flask 是一个轻量级的 Python web 框架，其采用 jinjia2 作为模板引擎，采用 Werkzeug 作为 WSGI (Web Server Gateway Interface) 工具箱。它采用 [extensions](http://flask.pocoo.org/extensions/) 的模式来丰富其功能，让用户可以依照自己的需要选择——甚至可以自己开发第三方插件。

## 2 从搭建博客的角度说起

要构建一个个人博客——也就是比较简单的个人主页，我们需要三样东西：一台常年运行的服务器，一个供用户访问的地址 URL ，提供内容的 WEB 程序。

前两样东西一般通过购买获得（当然第三样也可以购买或者使用别人搭建好的开源程序），WEB 程序书写则是我们现在的目标。我们知道，呈现给用户（客户端）的内容是 HTML+CSS+JavaScript 的组合，而 Flask 则是帮助我们在后台控制用户访问的工具。

Flask 作为一个轻量级的 Python WEB 框架，其核心只提供了最简单的功能，包括：

1. 路由 route ：通过装饰器将 URL 与视图函数绑定；
2. 调试 debug ：反馈程序中的错误并指出问题的代码；
3. Web 服务器网关接口 WSGI ：一种调用约定，用于 Web 服务器将请求转发到程序中；
4. 模板引擎 jinja2 ：对模板进行渲染。

其中，路由、调试和 WSGI 子系统均由 [Werkzeug](http://werkzeug.pocoo.org/) 提供。

至此，我们已经可以写出最简单的 Flask 程序：

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return "hello world"

if __name__ == "__main__":
    app.run()
```

但对于一个完整的博客而言，这样简单的功能显然难以满足我们的需求，幸好，我们可以借助于 Flask 丰富的第三方拓展完善网页的功能。

1. [Flask-WTF](https://flask-wtf.readthedocs.io/en/stable/): Web 表单的处理；
2. [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/2.3/): 数据库操作与访问；
3. [Flask-Mail](https://pythonhosted.org/Flask-Mail/): 电子邮件支持；
4. [Flask-Login](https://flask-login.readthedocs.io/en/latest/): 用户认证管理；
5. [Flask-Markdown](https://pythonhosted.org/Flask-Markdown/): 为博客提供 Markdown 相关支持；
6. [Flask-WhooshAlchemy](https://flask-whooshalchemy.readthedocs.io/en/latest/): 使用 Whoosh 实现 Flask-SQLAlchemy 模型的全文检索；
7. [Flask-Sitemap](https://flask-sitemap.readthedocs.io/en/latest/): 用于生成网页站点图；

更多有关 Flask 第三方库的推荐可参考 Awesome Flask：<https://github.com/humiaozuzu/awesome-flask>.

## 3 一些套路

### 3.1 虚拟环境

一般而言，使用 Python 做一些项目时，都会为该项目创建单独的虚拟环境，以避免包的混乱及版本的冲突，保持全局 Python 解释器的干净整洁的同时，更加便于使用（在虚拟环境中，不需要管理员权限）。

创建虚拟环境的 Python 包可以用 virtualenv, pyvenv, pipenv 等，以 virtualenv 为例创建项目所需的环境：

```bash
$ mkdir projectname
$ cd projectname

# 创建名为 venv 的虚拟环境（名称无强制要求）
$ virtualenv venv

# 激活虚拟环境
$ source venv/bin/activate
# Windows 环境下被激活
$ venv\Scripts\activate

# 安装 Flask
(venv) $ pip install flask

# 退出环境
$ deactivate
```

在 Windows 系统中的命令和类 UNIX 系统中不一致，\ / 之间的区别常让人头大， [cmder](http://cmder.net/) （可以在 Windows 系统中使用常用的 bash 命令）可以帮你解决大部分问题。

### 3.2 运行参数

假定你需要运行的 flask 程序为 `manage.py`，在测试环境中的运行命令为：

```bash
(venv) $ python manage.py runserver
```

一些常用的可选参数：

* -h, --help: 显示帮助信息并退出。
* -t HOST, --t HOST: 允许访问的 IP 地址，默认情况下只允许本地主机通过 `http://127.0.0.1:5000/` 进行访问；通过添加参数 `--host 0.0.0.0` ，任意计算机可通过本地主机外网 IP 地址的 5000 端口访问。
* -p PORT, --port PORT: 指定访问端口，默认为 5000 。
* -d, --no-debug: 不启用调试模式。

当然，这些只是在开发和测试过程中所运行的方式。一般而言，会为程序设定 `开发、测试、生产` 三种运行环境，**程序运行中所需要用到的一些变量从环境中读取**。生产环境中的运行命令会在后续说明中单独介绍。

### 3.3 数据保存

网页程序一般分为两部分，一部分是供运行的程序 ~ 如我们使用 Flask 写的程序，另一部分是网页的内容 ~ 如「用户的数据，文章，图片等」，一般使用数据库存储。

对于 Flask 程序而言，可使用 Git 进行管理 ~ 即购买的服务器既是「网页服务器」又是「Git服务器」，以便于版本控制及网页迁移。

对于网页内容，一般使用数据库进行存储（在 Flask 程序中使用 `flask-sqlalchemy` 来进行操作），使用数据库进行备份以保存数据。

如此即可实现网页的快速迁移，也可较好的对数据进行保存 ~ 网页迁移时只需要部署环境即可。

## 4 参考资料

1. Flask 官方网站, <http://flask.pocoo.org/>. 2018/09/23.
2. [MiguelGrinberg](https://blog.miguelgrinberg.com/), 格林布戈, 安道. Flask Web开发:基于Python的Web应用开发实战[M]. 人民邮电出版社, 2015.
3. Awesome Flask, <https://github.com/humiaozuzu/awesome-flask>.
