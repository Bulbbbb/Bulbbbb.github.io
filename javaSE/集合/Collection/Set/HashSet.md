---
aliases: 
tags: 
---

## 特点
+ _无序。_
+ _允许 null 值，不允许重复（会覆盖）。_
+ _线程不安全。_
+ _HashSet 基于 HashMap 实现，实际上使用的是 HashMap 的 key 来存储元素。_

## 存取原理

_HashSet 基于 HashMap 实现，实际上使用的是 HashMap 的 key 来存储元素。所以 HashSet 的底层根跟 HashMap 是一样的，请参考 HashMap。_

_HashSet 的 value 部分存放的是一个 Object 对象 PRESENT。_

```java
private static final Object PRESENT = new Object();
```
