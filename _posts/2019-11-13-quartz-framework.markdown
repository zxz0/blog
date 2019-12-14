---
layout: post
title:  "粗观Quartz调度框架"
date:   2019-11-13 12:00:00 -0700
categories: Framework
tags: Quartz Scheduler
description: Quartz是干什么的，设计的大概思想，怎么用
---

有段Quartz有关的代码需要维护，做了一点research


- JobDetail，是Job信息的抽象
- Trigger，是触发器，很好理解
- Job，则是具体的job，execute()之中的方法是干实事（business logic）的

一个Job可以有很多trigger，一个trigger只能trigger一个job，这种松耦合，给予了scheduler很大的灵活性。

具体机制：当事件被触发的时候，Job就会被instantiate，然后执行execute()方法。也就是说，Job是stateless的。这样就带来的一个问题，Job可能要用到的数据该存在哪里？别慌，有JobDataMap，在JobDetail里面。  
Trigger也想传一些信息？Trigger也有JobDataMap。  
这个Map，就是JDK的Map接口的一个实现。

至于Job在execute里面如何access这些信息？execute方法，重载的execute方法有一个参数是context，里面可以拿到包括JobDetail，Trigger等等的一系列相关信息。

scheduler本身，去scheduler job的时候，必须有JobDetail和Trigger。  
定义JobDetail和Trigger的时候，要指定name和group，这个(name, group)的组合要是unique的。

总之，因为是在一个application内的，所以如果有集群，要么就关掉其他的只剩下一个，要么就用消息中间件。