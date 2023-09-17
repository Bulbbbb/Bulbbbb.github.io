---
aliases: 
tags: 
---

*在 Linux 中，多路复用 IO 有 3 中实现方法：*
+ *select*
+ *poll*
+ *epoll*

*linux 中提供了一个系统调用 select，来供开发者使用 select 多路复用：*
![[Pasted image 20230529185858.png]]
*该函数通过轮询同时监视多个文件描述符是否发生了读、写、异常三类 IO 事件。*
*最后返回发生 IO 事件的文件描述符数量，以及读事件、写事件、异常事件分别发生在哪些文件描述符中。*

## 使用

### 获取通道

```java
SocketChannel channel = new Socket().getChannel();
```

### 创建 Selector

```java
Selector open = Selector.open();
```

### 向 Selector 注册通道

```java
channel.configureBlocking(false);  
channel.register(selector, SelectionKey.OP_READ);
```

*注册到 Selector 的通道必须处于非阻塞模式，FileChannel 必须处于阻塞模式，所以不能注册到 Selector。*

*register() 方法的第二个参数表示注册到 Selector 的 Channel 对什么事件感兴趣*
+ *SelectionKey.OP_READ：读事件*
+ *SelectionKey.OP_WRITE：写事件*
+ *SelectionKey.OP_CONNECT：连接事件*
+ *SelectionKey.OP_ACCEPT：都事件*
*如果 Channel 不止对一件事情感兴趣，可以通过”位或“运算符连接：*

```java
channel.register(selector, SelectionKey.OP_READ|SelectionKey.OP_WRITE);
```

### 轮询

```java
//循环等待IO事件  
while (true) {  
	//操作系统轮询，阻塞直到有IO事件发生，方法返回值表示有多少通道就绪  
	int select = selector.select();  
	// 获取触发事件的SelectionKey集合  
	Set<SelectionKey> selectedKeys = selector.selectedKeys();  
	Iterator<SelectionKey> iterator = selectedKeys.iterator();  
	while (iterator.hasNext()) {  
		SelectionKey key = iterator.next();  
		if (key.isAcceptable()) {  
			// 接收事件处理逻辑  
		} else if (key.isConnectable()) {  
			// 连接事件处理逻辑  
		} else if (key.isReadable()) {  
			// 读事件处理逻辑  
		} else if (key.isWritable()) {  
			// 写事件处理逻辑  
		}  
	}  
}
```

> [!note] *在日常使用过程中，通常开启一个线程专门进行轮询操作，而不是在主线程中轮询，轮询的线程叫做 Acceptor。*

## reactor 模型

*在网络 IO 设计中 reactor 是一种经常使用的设计模式，具体看一下下面这篇文章：*
*[一文搞懂Reactor模型与实现)](https://juejin.cn/post/7210375522512666679)*