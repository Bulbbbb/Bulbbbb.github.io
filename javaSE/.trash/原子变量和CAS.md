---
aliases: 
tags: 
---
_对于非原子操作需要使用 synchronized 修饰临界区代码，但是代价太高。_
_Java 中提供如下原变量来代替 synchronized：_
+ _AtomicBoolean_
+ _AtomicInteger_
+ _AtomicLong_
+ _AtomicReference_
+ _AtomicLongArray、AtomicReferenceArray_
+ ……
_上面原子类中包含一些原子方式实现组合操作的方法。_

_这些原子方式实现的方法都依赖于下面方法：_

```java
public final boolean compareAndSet(int expect, int updated）
```

_我们简称为 CAS。_
_该方法有两个参数 expect 和 update，以原子方式实现了如下功能: 如果当前值等于 expect，则更新为 update，否则不更新，如果更新成功，返回 true，否则返回 false。_
