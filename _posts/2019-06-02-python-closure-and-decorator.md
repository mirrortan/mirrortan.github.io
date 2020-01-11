---
layout: post
title: Python 中的闭包和装饰器
categories: Python
tags: [Closure, Decorator]
---

函数的装饰器可以以某种方式增强函数的功能，如在 Flask 中可使用 `@app.route('/')` 为视图函数添加路由，是一种十分强大的功能。在表现形式上，函数装饰器为一种嵌套函数，这其中会涉及到闭包的概念。而在嵌套函数之间，外部函数中的变量相对于内部函数而言为自由变量，使用时可能需要借助于 `nonlocal` 关键字进行声明。

## 1 nonlocal 声明

按变量的作用域进行分类，Python 中的变量可分为「全局变量」、「局部变量」以及「自由变量」。一般而言，Python 中使用变量前不需要声明变量，但假定在函数体中赋值的变量为局部变量 ~ 除非显示使用 `global` 将在函数中赋值的变量声明为全局变量！

而自由变量则是存在于嵌套函数中的一个概念 ~ 定义在其他函数内部的函数被称之为嵌套函数 `nested function` ，嵌套函数可以访问封闭范围内（外部函数）的变量。嵌套函数不可以在函数外直接访问。

在Python中，非本地变量默认仅可读取，在修改时必须显式指出其为非本地变量 ~ 自由变量 `nonlocal`，全局变量 `global`。

```python
>>> ga = 1
>>> def func():
...     nb = 2
...     def inner():
...         ga += 1
...         nb += 2
...         print('ga is %s, and nb is %s' % (ga, nb))
...     return inner
...
>>> test = func()
Traceback (most recent call last):
...
UnboundLocalError: local variable 'ga' referenced before assignment
```

未加入全局变量和自由变量声明时且使用赋值操作时，`inner` 函数的变量 `ga, nb` 默认为局部变量，会报错；如注释掉 `ga += 1` 后同样会报错：

```python
Traceback (most recent call last):
...
UnboundLocalError: local variable 'nb' referenced before assignment
```

可行改写如下：

```python
>>> ga = 1
>>> def func():
...     nb = 2
...     def inner():
...         global ga
...         nonlocal nb
...         ga += 1
...         nb += 2
...         print('ga is %s, and nb is %s' % (ga, nb))
...     return inner
...
>>> test = func()
>>> test()
ga is 2, and nb is 4
>>> test()
ga is 3, and nb is 6
```

通过显示声明 `ga, nb` 分别为「全局变量」和「自由变量」，此时如预期运行！

## 2 闭包

函数内的函数以及其自由变量形成闭包。也即闭包是一种保留定义函数时存在的自由变量的绑定的函数 ~ 这样在调用函数时，绑定的自由变量依旧可用。

闭包可以避免全局变量的使用以及提供某种形式的数据隐藏。当函数中的变量和函数较少且其中某个功能常用时，使用闭包来进行封装。当变量和函数更加复杂时，则使用类来实现。

```python
# 计算移动平均值的函数
def make_averager():
    series = []

    def averager(new_value):
        series.append(new_value)
        total = sum(series)
        return total/len(series)

    return averager
```

那么此时，`make_averager()` 函数的第二行 `series = []` 到第七行 `return total/len(series)` 为闭包，变量 `series` 为 `averager()` 函数中的自由变量！

```python
# avg 为一个 averager 函数对象 ~ 含自由变量的绑定
>>> avg = make_averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11
# 创建另一个 averager 函数对象
>>> avg2 = make_averager()
>>> avg2(1)
1.0
>>> avg2(18)
9.5
# 查看 avg, avg2 自由变量中保存的值
>>> avg.__closure__[0].cell_contents
[10, 11, 12]
>>> avg2.__closure__[0].cell_contents
[1, 18]
```

函数对象通过 `__closure__` 属性——返回 `cell` 对象元祖（函数中有多少嵌套函数则该元祖的长度有多长），生成该对象的函数被称之为闭包函数。

`func.__closure__[0].cell_contents`: 访问存储在 `cell` 对象中值。

## 3 Python Decorators

装饰器本身是一个可调用的对象 ~ 函数或类，其参数为另一个函数（被装饰的函数）。装饰器可能会处理被装饰的函数（如添加一些功能）然后将之返回，或者将之替换为另一个函数或可调用对象。这也被称之为元编程 `metaprogramming` —— 在编译时改变函数功能。

```python
>>> def make_pretty(func):
...     def inner():
...         print("I got decorated!", end='\t')
...         func()
...     return inner
...
>>> def ordinary():
...     print("I am ordinary!")

# 用 make_pretty 函数装饰 ordinary 函数
>>> pretty = make_pretty(ordinary)
>>> pretty()
I got decorated!  I am ordinary!
```

可以作为装饰器的函数内部都有嵌套的功能函数（用以实现主要功能），并返回内部的嵌套函数。

```python
@make_pretty
def ordinary():
    print("I am ordinary!")

# 等价于
def ordinary():
    print("I am ordinary!")
ordianry = make_pretty(ordinary)
```

`make_pretty(func)` 是一个最简单的装饰器，它接受一个函数为其参数；内部定义了一个 `inner()` 函数 ~ 输出 "I got decorated!    " 后执行被装饰函数（此时 `func` 为 `inner` 闭包中的自由变量）；然后返回内部函数 `inner`。

此时，对于被装饰的函数 `ordinary` 而言，此时是 `inner` 的引用：

```python
>>> ordinary()
I got decorated!  I am ordinary!
>>> ordinary
<function make_pretty.<locals>.inner at 0x10aeaa1e0>
```

除了最简单的装饰器之外，还可以将多个装饰器叠放使用以及对装饰器参数化：

### 3.1 叠放装饰器

```python
def star(func):
    def inner(*args, **kwargs):
        print('*' * 30)
        func(*args, **kwargs)
        print('*' * 30)
    return inner

def dollar(func):
    def inner(*args, **kwargs):
        print('$' * 30)
        func(*args, **kwargs)
        print('$' * 30)
    return inner

@star
@doller
def printer(msg):
    print(msg)

printer("Hello world!")

# 结果如下
'''
******************************
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Hello world!
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
******************************
'''

# 等价于
def printer(msg):
    print(msg)
printer = star(dollar(printer))
```

多个装饰器一同使用相当于逐层嵌套，最上方的为最外层的函数，调用时最先开始，且最晚结束。

### 3.2 参数化装饰器

Python 中将被装饰函数作为参数传递给装饰器函数。此外，我们可以创建一个装饰器工厂函数（返回装饰器的函数），把参数传递给它，再应用于要装饰的函数上：

```python
'''
Fluent Python 示例 7-23
https://github.com/fluentpython/example-code/blob/master/07-closure-deco/registration_param.py
'''
registry = set()

def register(active=True):
    def decorate(func):
        print('running register(active=%s)->decorate(%s)'
              % (active, func))
        if active:
            registry.add(func)
        else:
            registry.discard(func)
        return func
    return decorate

@register(active=False)
def f1():
    print("running f1()")

@register()
def f2():
    print("running f2()")

def f3():
    print("running f3()")
```

将上述代码保存至 registration_param.py 模块中，导入时得到结果如下：

```python
>>> import registration_param
running register(active=False)->decorate(<function f1 at 0x103ae0268>)
running register(active=True)->decorate(<function f2 at 0x103ae00d0>)
>>> registration_param.registry
{<function f2 at 0x103ae00d0>}
```

**注意**：函数装饰器在被导入模块时立即执行，而被装饰的函数只有在明确调用时运行。

### 3.3 常用的装饰器

Python 内置了三个装饰器，分别为：`property, classmethod, staticmethod`

- `property` 装饰器可用于设定类中的私有变量；
- `classmethod` 用于设定类方法；
- `staticmethod` 用于设定类中的静态方法。

此外，常用的装饰器还有 `functools` 模块中的 `wraps, lru_cache, singledispath`:

- `functools.wraps()`：保留被修饰函数原有的一些属性，如 `__name__, __doc__`；
- `functools.lru_cache()`：可把耗时的函数结果保存起来，避免传入相同的参数重复计算 ~ 可用于优化递归算法；
- `functools.singledispath()`：会把修饰的普通函数变为泛函数。

## 4 参考资料

1. [Programiz Home](https://www.programiz.com/), Parewa Labs Pvt. Ltd, [Python Decorators](https://www.programiz.com/python-programming/decorator), 2019/06/01.
2. Ramalho, Luciano. Fluent Python: clear, concise, and effective programming. " O'Reilly Media, Inc.", 2015.
