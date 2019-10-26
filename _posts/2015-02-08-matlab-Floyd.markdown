---
layout: post
title:  "matlab Floyd最短路计算"
date:   2015-02-08 22:17:55 +0800
categories: Algorithm
tags: Algorithm cat Bash Command Ops
description: 用matlab实现Floyd最短路计算
---
```matlab
clear;clc;
n=31; a=zeros(n);
a(1,2)=124;a(1,3)=187;a(1,4)=182*1.5;
...
a(29,30)=68*2;a(29,31)=41*2;
a=a+a'; M=max(max(a))*n^2; %M为充分大的正实数
a=a+((a==0)-eye(n))*M;
path=zeros(n);
b=a;
ccase=xlsread('各地区累计病例.xls');
for i=1:n
  b(:,i)=b(:,i)*ccase(i);
end
%a, path
```
计算所有点之间的最短距离，存于a，路径存于path。