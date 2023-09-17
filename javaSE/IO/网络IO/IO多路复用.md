---
aliases: 
tags: 
---
*多路复用的逻辑就是将轮询有操作系统来执行，为什么要由操作系统来完成，轮询的时候要执行系统调用，操作系统来做的话就不需要进行用户态和内核态的切换*

*当然我们可以另开一个线程来轮询，也不会影响到 listenerSocket 来接收请求，但是效率太低了。*

*我们把 listenerSocket 注册到多路复用器上，多路复用器用来监听 listenerSocket 上的 accept 事件。*
*然后我们定期轮询注册到多路复用器上的 listenerSocket，如果有请求要来连接，多路复用器被触发，然后我们拿到请求的 clientSocket，并且把 clientSocket 也注册到多路复用器上，不过我们注册的是读事件。*

*然后进行下一次轮询，现在轮询的就是两个 socket 了，轮询的是 listener 的 accept 事件和 clientSocket 事件的读事件。*
*当 clientSocket 从不可读变为都的时候，读事件被触发（边沿触发）*

![[Pasted image 20230826144624.png]]

## select、poll、epoll

*事实上，io 多路复用有 3 种类型，select、poll、epoll，Java 中的 nio 使用的是 epoll*
*并且 epoll 也是效率最好的一种方式*

*select、poll 并没有多大的区别，他们两个的运行逻辑是，用上一节的代码举例，我们把 ClientSocket 加入到 list 中 (不过 select、poll 使用的并不是 list，他们两的区别就在这里)，然后调用 select 和 poll 函数的时候，把这里 list 作为参数传递进入*

*也就是说 select、poll 每次轮询都需要将 socket 拷贝到内核中*
*与我们的实现相比，select、poll 只需要切换一个用户态和内核态，不需要每查询一个 socket 都进行内核态和用户态切换。*


*而 epoll 的逻辑大不相同，上面的代码就是 epoll 的逻辑，我们只需要将 socket 注册到多路复用器上，而不用每次都将 socket 拷贝到内核态*
*并且 epoll 会只返回被触发的 socket，而 select 和 poll 会返回全部的 socket，我们需要进一步轮询去发现那个 socket 被处罚。*

*epoll 使用红黑树来存储注册到多路复用器上的 socket*
*并且使用链表来存储就绪的 socket*
*每次获取的时候，返回就绪链表上的 socket。*

*epoll 中的 socket 会被事件触发*
+ *边沿触发*
+ *水平触发*

### epoll 的实现原理

*用 epoll_create 后，在内核中分配一个 eventpoll 结构和代表 epoll 文件的 file 结构，并且将这两个结构关联在一块，同时，返回一个也与 file 结构相关联的 epoll 文件描述符 fd*

*向 epoll 注册 socket 的时候：*
1. *创建一个 epitem， 加入到红黑树*
2. *添加等待时间到 socket 的等待队列中，设置回调函数*

*回调函数检查 socket 的触发事件与注册事件的交集，如果由交集则加入到 eventpoll 的就绪队列中*

*调用 wait 时：*
*检查就绪队列，然会队列的 size。*
