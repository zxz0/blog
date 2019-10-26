---
layout: post
title:  "【转】String 字符串常量池"
date:   2019-05-20 12:00:00 -0700
categories: Java
tags: Java JVM_Run-time_Data_Area String Memory_Management
description: JVM运行时数据区，以及对于String的内部处理
---
### 摘抄自：
- [String：字符串常量池](https://segmentfault.com/a/1190000009888357)
- [String s=new String（“abc”）创建了几个对象](https://blog.csdn.net/Liu_Fangyuan/article/details/80782302)

提高性能和减少内存开销：
- 为字符串开辟一个字符串常量池，类似于缓存区
- 创建字符串常量时，首先检查字符串常量池是否存在该字符串
- 存在该字符串，返回引用实例，不存在，实例化该字符串并放入池中
- eg. String str1 = “hello”; String str2 = “hello”; str1 == str2 // true

运行时数据区：
- 堆区：
	- 储存对象，每个对象都包含一个与之对应的class
	- JVM只有一个heap，被所有线程共享
	- 堆中不放基本类型和对象引用，只放对象本身
	- 对象由垃圾回收器负责回收，大小和生命周期不需要确定
- 方法区：
	- 静态区，和堆一样，被所有线程共享；
	- 整个程序中永远唯一的元素，如class，static变量
	- （字符串常量池在这里！！）（所有常量都储存在常量池中）
- Java栈区（栈）：
	- 每个线程包含一个栈，只保存基础数据类型的对象和自定义对象的引用（非对象本身）
	- 每个栈中的数据（原始类型和对象引用）都是私有的
	- 3部分：基本类型变量区，执行环境上下文，操作指令区（存放操作指令）
	- （变量和引用储存在这里）
		- eg. `public static x = 1;` x是在栈中的引用，指向常量池里面的1

eg. 
```java
String str4 = new String("abc"): // new String 创建对象 所以会放东西到堆
```

1.	在常量池中查找是否有“abc”对象
	1.	有则返回对应的引用实例
	2.	没有则创建对应的实例对象（创建对象1）
2.	在堆中 new 一个 String("abc") 对象（创建对象2）
3.	将对象地址赋值给str4,创建一个引用（引用实例 or 对象那里）

通过new操作符创建的字符串对象不指向字符串池中的任何对象，但是可以通过使用字符串的intern()方法来指向其中的某一个。java.lang.String.intern()返回一个保留池字符串，就是一个在全局字符串池中有了一个入口。如果以前没有在全局字符串池中，那么它就会被添加到里面  
eg.  
```java
// Create three strings in three different ways.
String s1 = "Hello";
String s2 = new StringBuffer("He").append("llo").toString();
String s3 = s2.intern();

// Determine which strings are equivalent using the ==
// operator
System.out.println("s1 == s2? " + (s1 == s2)); // false
System.out.println("s1 == s3? " + (s1 == s3)); // true
```

字符串对象内部用数组储存：
```java
String m = "hello,world";
String n = "hello,world";
String u = new String(m);
String v = new String("hello,world");
```

1.	会分配一个11长度的char数组，并在常量池分配一个由这个char数组组成的字符串，然后由m去引用这个字符串
2.	用n去引用常量池里边的字符串，所以和m引用的是同一个对象
3.	生成一个新的字符串，但内部的字符数组引用着m内部的字符数组
4.	同样会生成一个新的字符串，但内部的字符数组引用常量池里边的字符串内部的字符数组，意思是和u是同样的字符数组
```java
System.out.println(m == n); //true  m和n是同一个对象
System.out.println(m == u); //false m,u,v都是不同的对象
System.out.println(m == v); //false
System.out.println(u == v); //false 
```
m,u,v,n但都使用了同样的字符数组，并且用equal判断的话也会返回true