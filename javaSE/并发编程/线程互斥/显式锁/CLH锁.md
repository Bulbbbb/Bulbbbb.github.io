---
aliases: 
tags: 
---
*CLH 锁是一种基于队列的自旋公平锁，由于是 Craig、Landin 和 Hagersten 三位大佬的发明，因此命名为 CLH 锁。*


*==自旋锁==是指，当一个线程尝试获取某个锁时，如果该锁已被其他线程占用，就一直循环检测该锁是否被占用，而不是进程线程挂起或者休眠状态。所以自旋锁会一直占用 CPU。自旋锁的底层是基于 CAS 机制的。*

*与此相反，如果锁被其他线程占用是，该线程进入挂起或者休眠状态，则该所被称为==互斥锁==。与自旋锁相比，此时 CPU 可以空闲下来去运行其他线程。线程的状态切换以及线程的调度都需要 os 的参与，会消耗大量的资源。*

*==公平锁==是指，多个线程按照申请的顺序获得锁，先申请的线程总是先获取到锁。*
*==非公平锁==是指，每个线程获取锁的顺序是随机的，并不会遵循先来先得的规则，所有线程会竞争获取锁。*

## CLHLock

*公平锁有许多实现方式，具体的细节需要看具体的实现方式。*
*CLH 锁就是一种自旋公平锁，今天主要学习 CLH 锁。*

*CLH 锁是通过队列来实现的一种自旋公平锁，具体来说：*

*CLH 在声明的时候会初始化三个变量 curNode、predNode、tailNode，这三个变量都是 CLHNode 类型，其中有一个 boolean 类型的属性 locked，默认为 false，表示没有获取到锁或者锁已释放，true 表示获取到锁或者自旋等待。*
*其中 curNode 与 predNode 是 ThreadLocal 变量，每个线程中都有自己的一份。*
*tailNode 是属于 CLHLock 对象的，是所有线程共享的。*

> [!note] *ThreadLocal 变量在主线程中声明，此后每创建一个线程都会拷贝一份主线程的 ThreadLocal 变量。也就说每一个线程内都有自己独自的 ThreadLocal 变量。*

*curNode 是用于表示当前线程的结点，predNode 是用于表示前一个线程的结点（直线前一个结点的 curNode）。*
*CLH 初始化时会创建一个 curNode 用来表示当前线程，predNode 指向 null，以及一个 tailNode。*

> [!note] *不要怀疑 CLH 初始化时并没有线程申请 CLH 锁为什么会创建一个 curNode。因为并不是只有申请锁的线程才会加入 CLH 队列中，而是所有的线程都会在 CLH 队列中用 curNode 表示。但是只有申请加锁的线程才会加锁 CLHLock 的队列。*

![[CLH队列的初始化.excalidraw.png|CLHLock初始化时的状态|400]]

### lock

*下面说一下加锁的过程：*
*主线程中创建了一个 CLHLock，此时在主线程中创建 thread-1。*
*thread-1 中会有 curNode 和 predNode，但没有 tailNode，。*
*在主线程中创建 CLHLock 时的状态如上图所示。*

*当 thread-1 调用 lock() 申请锁的时候：*
1. *将 当前线程的 curNode 中的 locked 设为 true*
2. *将当前线程的 predNode 指向 tailNode 指向的结点，tailNode 总是执行尾结点的 curNode,所以也就是将 predNode 指向最后一个线程的 curNode*
3. *将 tailNode 指向当前线程的 curNode，当前线程成为了最后一个线程*
4. *通过 predNode.lock 进行自旋，等待 predNode.lock=false，也就是上一个线程释放锁，如果上一个线程没有释放锁，下一个线程就会永远在 lock 方法中循环，一旦释放锁，下一些线程则会跳出 lock 进入临界区。*

### unlock

*当 thread-1 调用 lock() 释放锁的时候：*
1. *将 当前线程的 curNode 中的 locked 设为 false，表示释放锁*


*更多细节看这里：[AQS的前菜—详解CLH队列锁_](https://blog.csdn.net/fengyuyeguirenenen/article/details/123856507)*