---
aliases: 
tags: 
---

> _ArrayDeque 是 Deque 的顺序存储结构的实现，可以同时作为队列和栈进行使用。_

> [!note] _ArrayDeque 的性能比 LinkedList 好。_

## 特点

+ _有序_
+ _不允许 null 值，允许元素重复_
+ _线程不安全_
+ _底层为数组_

> [!note] _ArrayDeque 底层通过数组实现，数组不能动态增减个数，因此删除元素时，ArrayDeque 会将该处元素置为 null，也就是说 ArrayDeque 中 null 是有特殊意义的，所有 ArrayDeque 不允许程序员向其中添加 null。_

## 实现

*ArrayDeque 时 Java 中队列的实现，队列是一种受限的线性表。*
*底层也是用一个 Object 数组 elements 来实现。*
*ArrayDeque 初始化的时候也要指定 elements 的容量，与 arrayList 不同的是，使用无参构造是直接对数组进行初始化，而不是 add() 时初始化，且默认容量为 16。*

*ArrayDeque 要注意的点相比于 ArryList 要注意入队和出队，因为入队和出队时要判满和判空，并且因为双指针的和循环队列的机制，判空和判满并不像 ArrayList 一样简单。*
*首先和 ArrayList 先判断再操作的逻辑不同的是，ArrayDeque 是先操作再判断。*
*操作完之后判空或者判满，如果满则扩容，如果取出元素为 null，则为空*
*其次，ArrayDeque 的判空条件为 head=rear，判满条件为 (rear+1)%max_size=head*

*ArrayDeque 再队满时进行扩容，扩容大小为 1.5 倍，这里需要注意的时，ArrayDeque 再扩容时，需要将元素转移到新数组。移动时并不是原封不动的移动到新空间就行了，因为扩容的空间需要移动到 rear 指针后面，所以可能需要移动元素。*

### 初始化

_ArrayDeque 使用数组实现队列，使用 head 和 tail 来记录队列中头和尾的索引。_
+ _head 记录队头元素的索引_
+ _tail 记录队尾元素的下一个索引，也就是下一个要插入位置的索引_

_因为全局变量会进行隐式初始化，所以 head 和 tail 指向的默认位置都是 0，即 head\=\=tail\=\=0。_

```java
transient Object[] elements;
transient int head;
transient int tail;
```

_对于 elements 数组的初始化，ArrayDeque 提供了三个构造函数：_
+ _无参构造_
+ _有参构造（参数为元素个数）_
+ _有参构造（参数为其他集合）_

_使用无参构造时，默认初始容量为 16_

```java
public ArrayDeque() {  
    elements = new Object[16];  
}
public ArrayDeque(int numElements) {  
    elements =  
        new Object[(numElements < 1) ? 1 :  
                   (numElements == Integer.MAX_VALUE) ? Integer.MAX_VALUE :  
                   numElements + 1];  
}
public ArrayDeque(Collection<? extends E> c) {  
    this(c.size());  
    copyElements(c);  
}
```

### 入队

_在《数据结构和算法》中总结过，入队之前要判空，出队之前要判满。判满的条件为：(rear+1)%QueueSize=front_

_ArryDeque 中判满的逻辑是一样的，不同的是循环队列的实现并不是通过对 QueueSize 取余实现的。_

```java
public void addLast(E e) {  
    if (e == null)  
        throw new NullPointerException();  
    final Object[] es = elements;  
    es[tail] = e;  
    if (head == (tail = inc(tail, es.length))) // 判满
        grow(1);  
}
```

_Java 中利用 inc 方法来实现增加时的循环。_
_inc 方法的功能是对 tail+1，如果超过数组的长度，则将 tail 置为 0，相当于重新指向队头。_

```java
static final int inc(int i, int modulus) {  
    if (++i >= modulus) i = 0;  // 先对 i+1，然后判断i是不是超过数组的长度
    return i;  
}
```

_通过上面入队的代码还可以发现，ArryDeque 并不是先判满再入队，而是先入队再判满，如果队满，则扩容。_

### 扩容

_ArryDeque 在队满时通过调用 grow() 方法进行扩容。_

```java
private void grow(int needed) {  
    // 旧长度  
    final int oldCapacity = elements.length;  
    int newCapacity;  
    // 如果 oldCapacity <64 ,扩容oldCapacity+2，否则扩容50%
    int jump = (oldCapacity < 64) ? (oldCapacity + 2) : (oldCapacity >> 1);  
    if (jump < needed  
        || (newCapacity = (oldCapacity + jump)) - MAX_ARRAY_SIZE > 0)   // 如果扩容大小必须要的扩容大小小，或者扩容后大于最大的可扩容长度，则重新计算扩容大小
        newCapacity = newCapacity(needed, jump);  
        
    final Object[] es = elements = Arrays.copyOf(elements, newCapacity);  // copy到新数组
    // 拷贝是时考虑read和head指针的关系
    if (tail < head || (tail == head && es[head] != null)) {  
        int newSpace = newCapacity - oldCapacity;  // 要插入的空间数
        System.arraycopy(es, head,  // 从head处
                         es, head + newSpace,  // 移动到head + newSpace处
                         oldCapacity - head);  // 移动长度为oldCapacity - head
        for (int i = head, to = (head += newSpace); i < to; i++)  
            es[i] = null;   // 将新增的空间置为null
    }  
}

private int newCapacity(int needed, int jump) {  
    final int oldCapacity = elements.length, minCapacity;  
    if ((minCapacity = oldCapacity + needed) - MAX_ARRAY_SIZE > 0) {  
    // 需要长度为 oldCapacity + needed ，如果需要长度 大于 最大长度，则使用最大可扩容长度
        if (minCapacity < 0)  
            throw new IllegalStateException("Sorry, deque too big");  
        return Integer.MAX_VALUE;  
    }  
    // 如果 need > jump , 最终长度为 oldCapacity + needed
    // 如果 need < jump , 最终长度为 oldCapacity + jump
    if (needed > jump)  
        return minCapacity;  
    return (oldCapacity + jump - MAX_ARRAY_SIZE < 0)  
        ? oldCapacity + jump  
        : MAX_ARRAY_SIZE;  
}
```

_下面是队列扩容时，队列的几种情况，因为 ArrayQueue 中是先插入后判满，所以在扩容时，如果队满，则一定是 tail == head && es[head] != null。_

![[队列扩容的几种情况.excalidraw.png|380]]

### 出队

_双端队列的底层通过 pollForst() 函数出队。_

```java
public E removeFirst() {  
    E e = pollFirst();  
    if (e == null)  
        throw new NoSuchElementException();  
    return e;  
}
```

_pollFirst() 函数获取 head 指针指向的元素_
_与入队相同，出队时也没有先判空，而是获取元素后判空，_
_如果取出的元素为 null，则认为队列为空，并返回 null_
_如果取出的元素不为空，则返回该元素，并把头指针指向的元素置为 null，将 head 指针通过 inc 方法 +1_

```java
public E pollFirst() {  
    final Object[] es;  
    final int h;  
    E e = elementAt(es = elements, h = head);  
    if (e != null) {  
        es[h] = null;  
        head = inc(h, es.length);  
    }  
    return e;  
}
```

### 队头增加元素和队尾删除元素

_队头增加元素、队尾删除元素和队头删除元素、队尾增加元素的区别就是指针向左移动。_
_该操作是通过 dec() 方法实现的_
_如果左移后指针小于 0，则将指针重新赋值为 QueueSize-1_

```java
static final int dec(int i, int modulus) {  
    if (--i < 0) i = modulus - 1;  
    return i;  
}
```
