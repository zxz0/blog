---
layout: post
title:  "【转】Python结构化"
date:   2019-02-09 12:00:00 -0700
categories: Python
tags: Python Structure
description: Python工程的结构化建议
---
### 摘抄自：[结构化您的工程](https://pythonguidecn.readthedocs.io/zh/latest/writing/structure.html)
README.rst  
LICENSE # 许可证。参照[https://choosealicense.com/](https://choosealicense.com/)  
setup.py # 打包和发布管理  
requirements.txt # 根目录，应该指明完整工程的所有依赖包：测试，编译和文档生成。若无，则非必须  
sample/\_\_init\_\_.py  
sample/core.py # 核心代码，如果只有1个文件，直接写在根目录下  
sample/helpers.py  
docs/conf.py  
docs/index.rst # docs/ 参考文档  
tests/test_basic.py # 包的集合和单元测试  
tests/test_advanced.py # 只有一组测试例子：一个文件当中；逐步增加：目录下  

关于测试：需要引用源文件，所以做法：不默认在site-packages（the target directory of manually built python packages）中，而是：tests/context.py:
```python
import os
import sys
sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))

import sample
```

individual test module:
```python
from .context import sample
```

Makefile:
```sh
init:
    pip install -r requirements.txt
test:
    py.test tests
PHONY: init test
```

模块： library.plugin.foo 不要使用下划线命名空间，而是使用子模块（在library文件夹中找）  
`import mode`: 在调用目录下找modu.py，找不到就recursively在path(PYTHONPATH)中找，仍然找不到：ImportError异常。  
代码放在模块命名空间里，不用担心重载同名方法。（所以如果只是引入模块，还需要指明命名空间才可用）  
最好的做法（提升可读性）：
```python
import modu
x = modu.sqrt(4) # 使用时指明package
```
----
包：将模块管理机制扩展到一个目录上（目录扩展为包）。任意包含\_\_init\_\_.py文件的目录都被认为是一个Python包。  
\_\_init\_\_.py将集合所有包范围内的定义。import pack.modu之后，先在pack目录下寻找\_\_init\_\_.py文件，执行顶层语句。如此之后，modu.py内定义的所有变量、方法和类在pack.modu命名空间中即可看到。  
`import very.deep.module as mod` # 简写太深的包  
把有隐式上下文和副作用的函数与仅包含逻辑的函数(纯函数)谨慎地区分开。  

装饰器：装饰函数，分离概念和避免外部不相关逻辑“污染”主要逻辑。使用例：记忆化/缓存。具体例子：需要在一个table中储存一个耗时函数的结果，这个并不属于当前函数逻辑。
```python
def decorator(func):
  # 操作func语句
  return func

@decorator
def decorated():
    print 'Hello' 
# is the same as:
def say_hello():
    print 'Hello'
decorated = deocrator(say_hello)
```
----
上下文管理器：Python对象，为操作提供额外的上下文信息。with初始化，在with块中可调用。  
自己实现：  
- 用类：类首先被实例化，\_\_enter\_\_和\_\_exit\_\_将分别在with内开始前和后调用。\_\_enter\_\_的返回值在```as f```语句中被赋给f。
- 生成器：
  ```python
  from contextlib import contextmanager

  @contextmanager  # 自带的
  def custom_open(filename):
    f = open(filename)
    try:
      yield f # 将控制权返回给with。as f部分将yield的f赋值给f
    finally:
      f.close() # 无论是否有异常，都调用

  with custom_open('file') as f:
      contents = f.read()
  ```
封装逻辑大？类better；简单操作：函数better。

动态类型： 指针，可以改变reference，推荐良好的命名习惯：不重复对同一个变量名赋值。使用一个名字，重复赋值时一样要创建新的对象（对效率没有提升）。

可变和不可变类型：列表，字典——可变。其他不可变的，赋值时候都创建新变量并命名。  
-> 可变类型不“稳定”，不可作为字典的键使用。  
字符串不可变。组合：每部分放到一个可变列表，join起来更高效。列表推导也比循环append()快。  
  eg. 
```python
nums = map(str, range(20)) # returns a list of the results after applying the given function to each item of a given iterable. 列表推导：nums = [str(n) for n in range(20)]
print "".join(nums)
```
eg.
```python
foo = 'foo'
bar = 'bar'
foobar = foo + bar  # 好的做法 when creating a new string from a pre-determined number of strings：用预先确定数量的字符串创建一个新的字符串，用加法操作符更快
foo += 'ooo'  # BAAAAD!!
foo = ''.join([foo, 'ooo']) # 应该用join，添加到已存在字符串，或者动态combine这样
```
关于打印，最好的做法：
```python
foobar = '{foo}{bar}'.format(foo=foo, bar=bar) # best
```