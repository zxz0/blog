---
layout: post
title:  "Python的类成员变量默认初始值的坑"
date:   2019-02-02 02:29:45 +0800
categories: Python
tags: Python Class
description: 给Python类成员变量设置默认初始值时，如果使用缺省值初始化，所有实例的对应变量将会指向的是同一个地址
---
问题发现：一个循环内，缺省值初始化同名变量，其中的list成员不是空，会延续之前同名变量的值。
---
示例代码：
```python
# Define class
class Variant():
    # use
    def __init__(self, price = 500, description = 'default description', values = ['', '', '']):
        self.price = price
        self.description = description
        self.values = values

    def __str__(self):
        return 'price: {}, description: {}, values: {}'.format(self.price, self.description, self.values)

variant_list = []
# Create instance with same name iteratively
for i in range(3):
    current_variant = Variant()
    if i == 1:
        current_variant.values[2] = 'hello'
    current_variant.price = i
    current_variant.description = 'description of variant: {}'.format(i)
    variant_list.append(current_variant)

# Test results
for variant in variant_list:
    print(str(variant))
```
----
结果：所有实例的values列表值相同

原因：“可选参数默认值的设置在Python中只会被执行一次，也就是定义该函数的时候”如此使用缺省值初始化，list成员指向的是同一个list（地址），如果只是修改其中一个元素（而不是赋值新的list开辟新内存），那么所有instance的list成员都会被修改。

解决方法：直接在构造方法中置为空（self.values = ['', '', '']），之后各个修改值

reference: [Python 类成员变量使用缺省值初始化时要注意的一个坑](https://blog.csdn.net/a462533587/article/details/80666444)
[Python程序员最常犯的十个错误](https://codingpy.com/article/top-10-mistakes-that-python-programmers-make/)