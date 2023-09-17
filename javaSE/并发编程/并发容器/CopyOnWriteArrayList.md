---
aliases: 
tags: 
---

## ArrayList 的线程安全问题

## CopyOnWrite

*当向 CopyOnWrite 容器中添加元素时，不直接往容器中添加，而是先将当前容器进行 Copy，复制出一个新的容器，然后往新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。*

## 相关操作

*CopyOnWriteArrayList 继承了 List 接口，所以说其基本操作是与 ArraysList 相同的，知识其方法的实现原理不同。*

*下面是 List 接口中的相关操作：*

![[List#^9ea51a]]

## 写

*借用 add() 方法来说明一下写操作的原理：*
1. *写时加锁（ReentrantLock）*
2. *写时赋值方式修改*

```java
public boolean add(E e) {  
	final ReentrantLock lock = this.lock;  
	lock.lock();  // 加锁
	try {  
		Object[] elements = getArray();  // Copy 原数组
		int len = elements.length;  
		Object[] newElements = Arrays.copyOf(elements, len + 1);  // 创建一个新数组，length+1
		newElements[len] = e;  // 插入新元素
		setArray(newElements);  // 将原引用指向新数组
		return true;  // 添加成功
	} finally {  
		lock.unlock();  // 释放锁
	}  
}
```

## 读

*读操作不加锁，和非线程安全的容器逻辑是相同的。*

```java
public int indexOf(Object o) {  
	Object[] elements = getArray();  
	return indexOf(o, elements, 0, elements.length);  
}
```

## 优缺点

1. *写时复制（读写分离）*
   *写时复制有什么优点？写时复制实现了读写分离，也就是读操和写操作并不在同一个 ArrayList 上进行。*
   *读写分离是有很多优点的，例如，数据库将读操作可写操作分离到不同的服务器就可以缓解服务器压力。*
   *CopyOnWriteArrayList 实现了将读和写分离到不同的对象上，有什么优点呢？*
   *读写分离可以提高效率，一个线程在写的时候，其他线程可以读，并且读操作可以并发进行。*
2. *弱一致性*
   *读写分离会导致数据的弱一致性，CopyOnWriteArrayList 只能保证数据的最终一致性，并不能保证实时一致性*

> [!note] *读写分离就是用线程的不安全性换取的效率。*

3. *内存占用大*
   *CopyOnWriteArrayList 在写操作时，内存中会同时存在两个对象，当集合中数据较小时可能并没有什么，但是当集合中数据较多时，会对内存造成较大压力。*

> [!note] *这里说一下，AyyayList 线程不安全有很多表现，基本上可以分为单一操作 (比如同时写) 导致的和复合操作导致的（同时读写）、迭代操作导致的，Vector 和 CopyOnWriteArrayList 基本上可以避免单一操作的不安全，但是并不能解决复合操作的不一致性。相比于 Vector，CopyOnWriteArrayList 的读写分离也会带来不安全性。也就说这两个并不是完全的并发安全的，在使用的使用千万要注意。*