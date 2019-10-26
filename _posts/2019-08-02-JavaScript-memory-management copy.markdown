---
layout: post
title:  "【转】JavaScript Memory Management"
date:   2019-08-02 12:00:00 -0700
categories: Front-End
tags: Front-End JavaScript Memory_Management
description: JavaScript的垃圾回收机制
---
### 总结自：
- [Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)

JS在objects创建的时候自动分配内存，不再被使用的时候释放（垃圾回收）。

memory life cycle：
1. 分配所需memory；
2. 使用分配的memory（读，写）；(explicit in all language)
3. 在不再被用到的时候，释放分配的内存。

1、3在low-level language中explicit，high-level中通常implicit.  
JS？在define的时候分配内存。比如function是一个callable object，会在定义function的时候被分配；function expressions也会分配一个object。  
一些function calls也会object allocation。比如`new Date()`（分配Date object），比如`document.createElement(‘div’)`（allocates a DOM element）；  
一些methods也会allocate new values / objects：`s.substr(0, 3)` （因为String是immutable）；`a.concat(a2);`  
Using values basically means reading and writing in allocated memory. This can be done by reading or writing the value of a variable or an object property or even passing an argument to a function.  
在不需要的时候释放内存：主要问题都在这一phase。最难的地方：确定何时分配的内存不再被需要。底层语言：developer手动决定；某些高级语言，如JS，GC：监控内存分配，确定何时某个被分配的内存块不再被需要，并且reclaim it. 只是approximation，undecidable。

Gabage Collection：
- reference（引用）：如果一个object has access to the another (implicitly or explicitly)，那么就说这个object reference（引用）另外一个。比如，一个JavaScript object有对其prototype的reference（implicit）以及对自己的properties values的（explicit）。
- Reference-counting garbage collenction: 最naive的垃圾回收算法。一个object是否还被需要 -> 一个object是否有任何object referencing it。如果0个references pointing to it，那么叫garbage，或者collectible。
	- 限制：circular reference：a reference b, b also reference a。但是永久都无法被回收。memory leak的常见原因。实际例子？dom元素reference自己的话，即使被从DOM tree中移除，也无法被回收。如果那个元素consume很多memory，那么永远无法回收则会造成browser速度很慢！
- Mark-and-sweep algorithm: 不再被需要 -> unreachable。assume roots，在JS中，是the global object. Garbage collector会定期从root开始，查找所有从root被referenc的，以及往深了找，被他们reference的。剩下的就是non-reachable的，会被回收。这样可以解决之前的circular reference的问题。但是手动释放内存就变难了：需要made explicitly unreachable（之前把所有reference置为空就可以了）。迄今（2019），不可能。
