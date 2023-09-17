---
aliases: 
tags: 
---
_我们知道 synchronized 可以实现线程同步，对于非原子操作需要使用 synchronized 修饰临界区代码，但是 synchronized 的代价太高。_

_Java 中提供如下原子类来代替 synchronized：_
+ _AtomicBoolean_
+ _AtomicInteger_
+ _AtomicLong_
+ _AtomicReference_
+ _AtomicLongArray、AtomicReferenceArray_
+ ……
_上面原子类中包含一些原子方式实现组合操作的方法。_

_这些原子方式实现的方法都依赖于下面方法，我们简称为 ==CAS==：_

```java
public final boolean compareAndSet(int expect, int updated）
```

_该方法有两个参数 expect 和 update，以原子方式实现了如下功能: 如果当前值等于 expect，则更新为 update，否则不更新，如果更新成功，返回 true，否则返回 false。_

*CAS 的操作过程如下：*
1. *比较 expect 和内存地址中的值*
2. *如果成功则将内存地址中值替换为 update*
3. *如果不相等则更新失败，并且重新执行步骤 1（自旋）*

*如果有其他进程在该进程提交之前修改了内存中的值，则此操作提交失败。*

*那有没有可能，在比较完成后，赋值之前内存中的值比修改了呢？在单核 CPU 中这种情况是不可能发生的，CAS 是一种原子操作，不能被打断。也就是说 CAS 在执行的时候是不可能切换到其他线程的。在多核中，原子操作需要上锁，内存是不可能同时被两个线程访问的。*

*那有没有可能，内存中的值被某一个线程修改了之后，又修改了回来，此时 CAS 操作判断 expect 和内存地址中的值仍然是相同的。有可能，这种问题我们称为 ==ABA 问题==。*

*也就是说普通的原子类会出现 ABA 问题。*
*解决 ABA 问题的方法是使用 AtomicStampedReference<\V>类代替普通的原子类。*
*AtomicStampedReference 通过在 CAS 操作中增加两个参数来防止 ABA 问题。*

```java
public boolean compareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp)
```

*在初始化 AtomicStampedReference 的时候传入一个初始的 stamp，在调用 compareAndSet() 的时候可以通过 getStamp() 来获取当前的 satmp，并且传入一个修改后的 stamp。*

*CAS 操作不仅比较要更新的值，还会比较传入的 stamp 和维护的 stamp 是否相等。*

*也就是不仅要通过 CAS 更新要更新的值，还要更新 stamp。*

```java
static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference(1,0);

Integer i1 = atomicStampedReference.getReference();
int s1 = atomicStampedReference.getStamp();
Integer i2 = i1 + 1;
int s2 = atomicStampedReference.getStamp() +1;
boolean result = atomicStampedReference.compareAndSet(i1,i2,s1,s2);
```