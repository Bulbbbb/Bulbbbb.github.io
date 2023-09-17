---
aliases: 
tags: 
---

### 总结

*AQS 是一个抽象类，提供了一系列方法，来解决并发编程中出现的问题，AQS 主要用来解决并发编程中的线程同步问题，和线程互斥问题。*

*对于互斥问题，解决的常用手段就是用一个共享变量和一个阻塞队列来实现，
线程加锁就是操作共享变量，如果可以操作，代表没有其他线程占有锁，则该线程继续往下运行。如果不能操作共享变量，则表示有线程占有该锁，该线程应该加入阻塞队列。*

*在 AQS 中，实现了上面的逻辑，提供了一个共享变量 state，并且可以使用内部类 Node 来构成阻塞队列。*

*AQS 中提供的加锁方法为 acquire()，在加锁时，首先尝试对共享变量 state + 1，这里可以通过控制 state 的取值范围来实现两种逻辑，如果 state 只能取 1 和 0，为独占模式，只允许一个线程占有锁，如果 state 取值大于 1，为共享模式，允许多个线程占有锁。*

*如果能操作，就继续往下运行，如果不能操作，AQS 中构建一个 Node 结点来代表当前线程，并将 Node 加入队尾。加入队尾后，并不立即阻塞，如果前继结点为 head 则循环等待，如果前继结点不为 head，则将前继结点状态设置为 SIGNAL，然后阻塞该线程。*

*同时 AQS 中提供了释放锁的方法 release(),release() 首先尝试获取锁，如果可以释放，还可以判断当前结点的状态是否为 SIGNAL，如果为 SIGNAL，还需要唤醒后继结点。*

*多线程过程中需要解决的另一个问题就是线程同步问题，使线程可以使用相互配合完成某些任务。*
*对于同步问题，常用的方法是使用条件变量和等待队列来实现。*
*常见的线程同步模型应该是生产者消费者模型，生产者生产的时候，如果队列为满，则应该到等待队列中等待，等生产者消费之后，应该唤醒等待队列上的生产者。*
*同样，如果队列为空，消费者消费的时候也应该进入等待队列中等待，等生产者生产之后唤醒等待队列上的消费者。*

*AQS 中同样提供了线程同步的方法，AQS 中有一个内部类 ConditionObject，代表条件变量，并别同样使用 Node 类来构建条件队列。*

*每一个条件变量 ConditionObject 都有自己的条件队列。*
*在检测到不满足条件时，应该构建一个条件变量，并将线程加入该变量的条件队列。也就是 new 一个 ConditionObject 对象，然后调用对象的 await() 方法。await() 方法中创建一个 Node 结点，代表当前线程，然后加入队尾。*
*在满足条件后，应该调用相应条件变量的 signal() 方法唤醒条件变量上的一个线程，或者 signalAll() 唤醒所有线程。*

### 详细

*在 Java 中，可重入锁，ReentrantLock 就是借助 AQS 实现的。*


*AbstractQueuedSynchronizer(AQS) 是一个抽象类，提供了一系列通用功能的实现来解决并发编程中出现的问题，来防止竟态条件问题和实现线程间的协作，是 JUC（java.util.concurrent）的基石。*

*[(68条消息) 大厂面试必会：AQS LockSupport Unsafe之间的关系_Hugo Lei的博客-CSDN博客](https://blog.csdn.net/hugo_lei/article/details/105813614)*
*[Java 并发高频面试题：聊聊你对 AQS 的理解？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/361477145)*

*AQS 中通过共享变量和同步队列来解决线程之间的互斥问题，通过条件变量和条件队列在解决线程之间的同步问题。*
![[Pasted image 20230507183801.png|400]]

*前面说解决竟态条件有两种方式，分别是互斥情况下和共享情况下，AQS 中提供的两种模式对应这两种情况：*
+ *独占模式*
+ *共享模式*
*在独占模式之下，同一时间只有一个进程可以访问临界区。*
*在共享模式之下，同一时间可以有多个进程可以访问临界区，但是只能读。*

*AQS 中的同步队列是基于 CLH 队列实现的，CLH 队列看这里 -->[[CLH锁]]。*
*AQS 中的同步队列对 CLH 队列进行了改进：*

*CLH 中提供了 curNode、predNode、tailNode 三个结点，curNode 和 predNode 为每个线程私有，通过 curNode 和 predNode 来实现单向队列，tailNode 为所有线程共享，指向队尾。*
*AQS 中每个线程都有一个结点，所有线程的结点都是共享的，通过结点中的 next 和 prev 指针来实现一个双向队列，通过 head 和 tail 指针指向队头和队尾。*

*CLH 中的 curNode 和 predNode 为 threadLocal，每个线程创建时都会创建，AQS 中代表线程的结点只有在线程指向 lock 时才会创建。*

*CLH 中通过结点中的 locked 变量来表示线程状态，由 true 和 false 两种， AQS 中使用 Node 结点中 waitStatus 用来表示线程状态，共有 5 种。*

| 状态名        | 描述                                                                                                                   |
| ------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 0             | 当一个 Node 被初始化的时候的默认值                                                                                     |
| SIGNAL(-1)    | 表示该节点正常等待                                                                                                     |
| PROPAGATE(-3) | 当前线程处在 `SHARED` 情况下，该字段才会使用。                                                                         |
| CONDITION(-2) | 该节点位于条件队列，当前线程阻塞在 condition，不能用于同步队列节点                                                     |
| CANCELLED(1)  | 值为 1，在同步队列中等待的线程等待超时或者被中断，从同步队列中取消等待。该状态下的节点不会进入其他状态，也不会再阻塞。 |

*CLH 种没有竞争到锁的线程会自旋等待，AQS 中使用自旋 + 阻塞 + 唤醒来代替纯自旋。*

## acquire()

*acquire() 是一个模板方法，其中包含了一个钩子方法，通常来说，借用 AQS 来实现琐时，在 lock 时，只需要重写钩子方法，并调用 acquire() 方法。*

*acquire 首先会调用 tryAcquire 获取锁。*

```java
public final void acquire(int arg) {  
	if (!tryAcquire(arg) &&acquireQueued(addWaiter(Node.EXCLUSIVE), arg))  
		selfInterrupt();  // 获取锁后，重新设置线程的中断标志，因为在申请锁时将中标标志重置了
}
```

*如果没有获得锁则首先调用 addWaiter() 函数为线程创建一个结点，并将新建结点插入队尾。*

```java
private Node addWaiter(Node mode) {  
	Node node = new Node(mode);  // 创建节点
	for (;;) {  // 循环执行下面操作
		Node oldTail = tail;  // oldTail指向尾结点
		if (oldTail != null) {  // 如果尾节点存在，队列不为空
		/*新建节点加入队尾*/
			node.setPrevRelaxed(oldTail);  // 将新建节点的prev指针指向oldTail
			if (compareAndSetTail(oldTail, node)) {  // 将tail指针指向新建节点
				oldTail.next = node;  // 将oldTail.next指向新建节点
				return node;  // 返回新建节点
			}  
		} else {   // 如果尾结点不存在
			initializeSyncQueue();  // 初始化队列，新建一个虚节点，head和tail指向虚结点
		}  
	}  
}
```

*在将代表该线程的结点插入队尾后，调用 acquireQueued() 方法使用循环不断为该节点去尝试获取锁，直到获取成功或者将线程阻塞。*

```java
final boolean acquireQueued(final Node node, int arg) {  
	boolean interrupted = false;  // 是否中断
	try {  
		for (;;) {  // 循环执行下面操作
			final Node p = node.predecessor();  // 获取前继结点
			if (p == head && tryAcquire(arg)) {  // 如果前继结点为头节点，尝试获取锁,成功后进入if方法体 （头节点为获取到锁的结点，证明该结点第一个排队，获取到锁说明占用锁的线程已经释放锁）
				setHead(node);  // 将当前结点设置为头节点
				p.next = null; // help GC  将上一个头节点的next指针设为null后就没有指针指向上一个头节点了，可以帮助GC
				return interrupted;   // 线程没有被中断
			}  
			if (shouldParkAfterFailedAcquire(p, node))   // 如果前继结点不为头节点，或者获取锁失败，判断该节点是否需要阻塞
			/*
			* 如果中断标志位false，interrupted 为 false
			* 如果中断标志位true，interrupted 为 true
			*/
				interrupted |= parkAndCheckInterrupt(); 
		}  
	} catch (Throwable t) {  
		cancelAcquire(node);  
		if (interrupted)  
			selfInterrupt();  
		throw t;  
	}  
}
```

*shouldParkAfterFailedAcquire() 函数用来检查线程是否应该阻塞。*
1. *如果前继结点 waitStatus 为 SIGNAL，则该线程应该阻塞，然会 true。*
2. *如果前继结点 waitStatus 为 Cancelled，则将删除前继结点一直到其前继结点状态不为 Cancelled，然后再下一次自选中判断是否应该阻塞。返回 false*
3. *如果前继结点不为 Cancelled 和 SIGNAL，则手动将状态调整为 SIGNAL，使线程在下次自旋中阻塞，返回 false。*

*综上所属，没有获得锁的线程，如果其前继结点不为头节点，通常情况下都会阻塞。*

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {  
	int ws = pred.waitStatus;  // 获取前继结点的waitStatus
	if (ws == Node.SIGNAL)  // 如果为SIGNAL
		return true;  // 可以阻塞
	if (ws > 0) {  // 如果前继结点状态大于0，Cancelled，再给一次获取锁的机会
	/*
	* 下面操作将会将队列中Cancelled状态的结点从队列中去掉
	*/
		do {  
			node.prev = pred = pred.prev;   // 将当前结点的prev指针指向pred节点的prev指针
		} while (pred.waitStatus > 0);  // 直到waitStatus<=0
		pred.next = node;  
	} else {   // 如果结点状态不为SIGNAL、Cancelled，再给一次获取锁的机会
		pred.compareAndSetWaitStatus(ws, Node.SIGNAL); // 将前继结点状态变为SIGNAL
	}  
	return false;  // 不可以阻塞
}
```

*parkAndCheckInterrupt() 函数将线程休眠，并检查线程的中断标志。*

```java
private final boolean parkAndCheckInterrupt() {  
	LockSupport.park(this);  // 将线程状态变为waitting
	// 检查中断标志，并清除中断标志，清除后，该线程只能被unpark唤醒，不能被interrupt唤醒，
	// 被interrupt唤醒的线程，如果再次阻塞，不能使用unpark唤醒
	return Thread.interrupted();  
}
```

*这里，必须使用 Thread.interrupted()，如果不使用 Thread.interrupted()，线程被唤醒后再次尝试获取资源，如果没有获取到，但是此时中断标志没有被清除，此线程不能被 park，若始终获取不到锁，将会在 acquireQueued 中循环，占用 CPU。*

*与同步队列相关的方法：*

| 方法名            | 描述                                       |
| ----------------- | ------------------------------------------ |
| addWaiter         | 创建节点并将节点添加到队尾，然后返回该节点 |
| enq               | 将指定节点添加到队列尾部                   |
| setHead           | 将指定结点设置为头节点                     |
| compareAndSetTail | 利用 CAS 机制，将指定节点设置为尾结点      | 

*在入队操作中，如果队列为空，则调用 initializeSyncQueue() 进行初始化，initializeSyncQueue() 中会创建一个新节点，并使 head 指针和 tail 指针指向这个结点。*

*需要注意的是，等待队列的头节点和尾结点只有在第一次入队时才会进行初始化，在 initializeSyncQueue() 中会创建一个==虚结点==，然后将 head 和 tail 指针指向这个虚结点。这个虚结点没有任何意义，也不代表一个线程。*
*然后，通过 addWaiter() 新建的结点或者通过 enq() 入队的结点将会作为 head 的后继节点，也就是等待队列中的第二个结点。*
*之后，tail 指向新建节点。*

```java
private final void initializeSyncQueue() {  
	Node h;  
		if (HEAD.compareAndSet(this, null, (h = new Node())))  
	tail = h;  
}
```

## release()

*release() 方法用来释放锁，在 unlock() 方法中调用,同样包含钩子方法 tryRelease()。*
*release() 方法会首先调用 tryRelease() 方法来尝试释放锁。*

```java
public final boolean release(int arg) {  
	if (tryRelease(arg)) {  // 释放锁成功
		Node h = head;  // 获取头节点
		if (h != null && h.waitStatus != 0) // 如果等待队列不为空，并且当前运行线程的waitStatus为SIHNAL则唤醒后继结点  
			unparkSuccessor(h);  // 唤醒
		return true;  // 释放成功
	}  
	return false;  // 释放失败
}
```

## 同步队列

*通过上面的 require() 和 release() 我们已经可以总结出，AQS 是如何通过同步队列和共享变量来实现互斥的。*
*AQS 内部维持一个等待队列，此队列为双向链表。*
*双链表中每一个结点代表一个线程：*
![[Node.excalidraw.png|100]]
*同时，AQS 内部维持一个共享变量 status。*
*在线程申请锁时，会通过 status 的值来判断当前线程是否可以持有锁，如果申请失败，将会创建一个结点并插入等待队列队尾。*

*等待队列将会在第一次入队操作时进行初始化，会创建一个空结点，然后将 head 指针和 tail 指针指向这个结点。*
![[Node初始化.excalidraw.png|100]]
*而第一次插入的结点将变为队列中第二个结点：*
![[Node插入.excalidraw.png|300]]
*处于等待队列中的线程将会自旋获取锁或者 park。*

*当获取锁后，将会将获取锁的线程结点从等待队列中删除，很简单，直接将该节点作为头节点即可。*

## 条件队列

*AQS 的内部有一个内部类 ConditionObject，实现了 Condition 接口。ConditionObject 类对象就是条件变量，ConditionObject 类内部维持了一个条件队列，通过条件队列实现了 await 和 signal 操作。*

*条件队列是一个单向队列，结点也是 Node 类型，使用 nextWaiter 指向后继结点。结点的 waitStatus 只能为 CANDATION 或者 CANCLED。*

*也就是每一个条件变量中都会有一个条件队列。*

*wait 操作会将结点加入条件队列，signal 操作会将结点从条件队列迁移到同步队列中并唤醒线程。*
*具体首先可以看一下显式条件的实现原理 ->[[并发编程/线程协作（同步）/显式条件/实现原理|实现原理|显式条件的实现原理]]。*

