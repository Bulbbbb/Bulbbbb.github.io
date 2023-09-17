---
aliases: 
tags: 
---
_在 List 接口中介绍过一次 LinkedList，这里的 LinkedList 和 List 接口下的 LinkedList 是同一个类。_
_通过阅读源码可以发现，LinkedList 同时实现了 List 接口和 Deque 接口，也就是说 LinkedList 可以通过作为 List 和 Deque 进行使用。_

> _LinkedList 作为 Deque 使用时，是 Deque 的链式存储结构的实现，==LinkedList 可以同时当作链表、队列和栈进行使用。==_
