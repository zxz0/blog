---
layout: post
title:  "【转】为何选择31做hashCode的mod？"
date:   2019-06-17 12:00:00 -0700
categories: Java
tags: Java HashCode
description: 31特殊的理由：不大不小，且可被JVM优化
---
### 摘抄自：
- [科普：为什么 String hashCode 方法选择数字31作为乘子](https://segmentfault.com/a/1190000010799123)

Java String类中的hashCode():
```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value; // String 内部维护的一个 char 类型数组
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
即为：s[0]\*31^(n-1) + s[1]\*31^(n-2) + ... + s[n-1]

原因：
1. 31: 质数，不大不小。质数可以降低哈希冲突；不大不小可以不太分散（overflow），也不太集中（冲突率上升）。奇数乘法overflow不会丢失信息：偶数的话，会：* 2 相当于shifting。
2. 31: 可以被JVM优化：31 * i = (i << 5) - i（移位和减法）