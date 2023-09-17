---
aliases: 
tags: 
---

## 特点

+ _有序_
+ _允许 null 以及元素重复_
+ _线程不安全_
+ _底层为双向链表_

> [!note] _LinkedList 的有序仅限于使用同一种插入方式进行插入，如果同时使用头插或者尾插可能会打破有序。_

## 原理

*LinkedList 是 Java 对顺序表的链式存储结构链表的实现。LinkedList 是一个双向链表。*

_LinkedList 的底层为双向链表，使用静态内部类 `Node<E>` 来表示链表的结点。_

```java
transient int size = 0;  
transient Node<E> first; // 头指针
transient Node<E> last; // 尾指针
private static class Node<E> {  
    E item;  
    Node<E> next;  
    Node<E> prev;  
  
    Node(Node<E> prev, E element, Node<E> next) {  
        this.item = element;  
        this.next = next;  
        this.prev = prev;  
    }  
}
```

> [!summary]+ _LinkedList 就是一个链表。_
