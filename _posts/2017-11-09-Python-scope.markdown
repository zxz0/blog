---
layout: post
title:  "Python的作用域陷阱"
date:   2017-11-09 15:59:35 +0800
categories: Python
tags: Python Scope Block Local
description: Python变量作用域（无block级，最低函数级，如JavaScript中的var）
---
问题发现：for循环内部的“局部”变量，在出现异常后，赋的变量仍有值可以输出。
每个循环都输出，发现没有重复。

结论：无for内的“局部”变量。
Python内的变量作用域，最小是以函数为单位。
之上是类，再是模块。

对于本问题，而输出没有重复，是因为出现异常后就跳出了赋值语句块，输出的设计不好。
实际上输出的值，是上次循环中赋的值。

延伸阅读：[Python变量作用域](https://blog.csdn.net/cc7756789w/article/details/46635383)