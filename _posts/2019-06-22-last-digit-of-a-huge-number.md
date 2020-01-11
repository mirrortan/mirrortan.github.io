---
layout: post
title: Last digit of a huge number
categories: [Pratice, Python]
tags: Algorithm
---

这是我在 [Codewars](https://www.codewars.com/) 中遇到的一个比较有意思的题目：求一个列表中元素累计乘方的最后一位数。

## 1 题干描述

For a given list [x1, x2, x3, ..., xn] compute the last (decimal) digit of x1 ^ (x2 ^ (x3 ^ (... ^ xn))).

E. g.,

last_digit([3, 4, 2]) == 1
because 3 ^ (4 ^ 2) = 3 ^ 16 = 43046721.

Beware: powers grow incredibly fast. For example, 9 ^ (9 ^ 9) has more than 369 millions of digits. lastDigit has to deal with such numbers efficiently.

Corner cases: we assume that 0 ^ 0 = 1 and that lastDigit of an empty list equals to 1.

This kata generalizes Last digit of a large number; you may find useful to solve it beforehand.

原题参见：<https://www.codewars.com/kata/5518a860a73e708c0a000027>.

简而言之，就是给定一个列表，求累计乘方的结果的个位数数值。

## 2 题目分析

对于这类题目，直接**硬算**显然是不可能的，题中给出的几个测试样例就过不了：

```python
# 样例
test_data = [
    ([], 1),
    ([0, 0], 1),
    ([0, 0, 0], 0),
    ([1, 2], 1),
    ([3, 4, 5], 1),
    ([4, 3, 6], 4),
    ([7, 6, 21], 1),
    ([12, 30, 21], 6),
    ([2, 2, 2, 0], 4),
    ([937640, 767456, 981242], 0),
    ([123232, 694022, 140249], 6),
    ([499942, 898102, 846073], 6)
]
```

题干其实已经给了我们提示，只要累计乘方的个位数值，那么显然应该存在某种规律，经过测试后发现，数值的幂次结果的个位数以 4 存在循环：

```python
def printer(numbers, numbers_power):
    print(' ', ' '.join(str(power) for power in range(len(numbers_power[0]))))
    for number in numbers:
        print(number, ' '.join(str(result)[-1] for result in numbers_power[number]))

if __name__ == "__main__":
    numbers = list(range(9))
    numbers_power = []
    for number in numbers:
        numbers_power.append([number**power for power in range(13)])
    printer(numbers, numbers_power)
```

将之保存为 demo.py 并运行得到如下结果：

```shell
$ python3 demo.py
  0 1 2 3 4 5 6 7 8 9 10 11 12
0 1 0 0 0 0 0 0 0 0 0 0 0 0
1 1 1 1 1 1 1 1 1 1 1 1 1 1
2 1 2 4 8 6 2 4 8 6 2 4 8 6
3 1 3 9 7 1 3 9 7 1 3 9 7 1
4 1 4 6 4 6 4 6 4 6 4 6 4 6
5 1 5 5 5 5 5 5 5 5 5 5 5 5
6 1 6 6 6 6 6 6 6 6 6 6 6 6
7 1 7 9 3 1 7 9 3 1 7 9 3 1
8 1 8 4 2 6 8 4 2 6 8 4 2 6
9 1 9 1 9 1 9 1 9 1 9 1 9 1
```

也即，除 0 次幂意外，`x ** n` 的个位数意思循环：`str(x ** n)[-1] == str(x ** (n+4))[-1]`！（整数 x 的 n 次幂的个位数值由 x 的个位数值的 n 次幂决定）

在这个题目中，我们将幂次运算的结果作为下一次运算的幂，而根据上一步我们得到幂次运算结果个位数值以 4 为循环的结果（0 次幂除外）。将 demo.py 的 printer 函数的最后一行改为：

```python
print(number, ' '.join(str(result % 4) for result in numbers_power[number]))
```

运行后得到的结果是：

```shell
$ python3 demo.py
  0 1 2 3 4 5 6 7 8 9 10 11 12
0 1 0 0 0 0 0 0 0 0 0 0 0 0
1 1 1 1 1 1 1 1 1 1 1 1 1 1
2 1 2 0 0 0 0 0 0 0 0 0 0 0
3 1 3 1 3 1 3 1 3 1 3 1 3 1
4 1 0 0 0 0 0 0 0 0 0 0 0 0
5 1 1 1 1 1 1 1 1 1 1 1 1 1
6 1 2 0 0 0 0 0 0 0 0 0 0 0
7 1 3 1 3 1 3 1 3 1 3 1 3 1
8 1 0 0 0 0 0 0 0 0 0 0 0 0
9 1 1 1 1 1 1 1 1 1 1 1 1 1
```

我们发现，`(x ** n) % 4` 稳定循环 ~ 0 次幂和 1 次幂除外 ~ 我们可以将大于 4 的幂次结果使用 `(x ** n) % 4 + 4` 来保持结果的一致性。

## 3 解决方案

综上，写出的代码如下：

```python
def last_digit(lst):
    # Your Code Here
    n = 1
    for x in reversed(lst):
        n = x ** (n if n < 4 else n % 4 + 4)
    return n % 10
```

最后，附上原题网址：<https://www.codewars.com/kata/5518a860a73e708c0a000027>。
