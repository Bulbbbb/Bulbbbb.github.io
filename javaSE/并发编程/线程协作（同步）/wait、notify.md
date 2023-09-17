---
aliases: 
tags: 
---

_wait 和 notify 相关的方法定义在 Object 类中，方法使用 final 修饰，无法重写。_

_wait 相关方法：_

```java
public final void wait() throws InterruptedException
public final native void wait(long timeoutMillis) throws InterruptedException;
public final void wait(long timeoutMillis, int nanos) throws InterruptedException
```

_notify 相关方法：_

```java
public final native void notify();
public final native void notifyAll();
```

_对象除了等待队列外，还有一个条件队列，该队列用于线程间的协作。_
_wait：将当前线程放到条件队列中并阻塞。_
_notify：从条件队列中选择一个线程，移出条件队列并唤醒。_

> [!note] *调用 wait 和 notify 的方法所在线程必须拥有此对象的锁，即必须使用 synchronized 进行修饰。*

```java
class TaskQueue {
    Queue<String> queue = new LinkedList<>();

	public synchronized void addTask(String s) {
	    this.queue.add(s);
	    this.notify(); // 唤醒在this锁等待的线程
	}
	
	public synchronized String getTask() {
	    while (queue.isEmpty()) {
	        this.wait();
	    }
	    return queue.remove();
	}
}
```

_在上面例子中，当执行 getTask() 时，如果队列为空，该线程进入条件队列等待。调用 getTask() 的对象，必然是获得锁的对象。_

_在 wait 后，该线程会释放锁，使其他线程有机会调用 addTask() 向队列中添加元素。所以调用 wait 的方法需要使用 synchronized 进行修饰。_

_在其他线程调用 addTask() 后，调用 addTask() 的线程使用 notify 唤醒正在等待的线程。_

_notify 并不会释放锁，也就是如果 addTask() 没有执行完，该对象的锁是不会被释放的。_

_addTask() 方法执行完，锁被释放，被 notify 唤醒的进程重新竞争锁，获得锁的线程从 wait 中返回，没有获得锁的进程此时会进入阻塞队列而不是留在等待队列。也就是说获得锁的进程从 wait 后的代码开始执行，没有获得锁的线程再获得锁后需要从方法的开头执行。并且有可能再次进入条件队列等待。_

> [!note] _进入 wait 的条件不能使用 if 语句进行判断，否则会出现虚假唤醒问题。_
> _假如 notify 唤醒三个线程，但是唤醒之后，调用 notify 的线程并没有释放锁，此时该线程删除了队列中元素，队列中为空。被唤醒的三个线程中有一个获取到锁此时从 wait 中返回，此时返回的线程从 if 语句中跳出再次获取元素，则会报错。_

### 如何实现两个对象交替打印？

*下面两个代码实现交替打印 A 和 B ，*
	*a 线程运行时拿到 Thread.class 的锁，然后 wait，注意 wait 的时候是释放锁的*
	*b 线程运行时拿到 Thread.class 的锁，然后打印 B，然后唤醒 a，wait*
	*然后 a 被唤醒后打印 A，然后唤醒 B，wait*

*需要注意的是，这里是先打印 A，然后打印 B。如果需要先打印 A，那就再 b 线程中打印 A，再 a 线程中打印 B。*
*不能先 start B，不然 B 拿到锁后，执行 notify，此时 B 并没有释放锁，所以 A 又被阻塞，然后打印 B，wait 之后，A 进入方法，然后 A wait，这个时候两个都 wait 了。*

*再一个：*
*调用 wait 的地方必须使用 synchronized 修饰，因为 wait 会释放锁*
*必须由加锁的对象调用 wait 和 notify，也就是 Thread.class*

```java
public class LoginController {  
public void print() {  
	Thread a = new Thread() {  
		public void run() {  
			synchronized (Thread.class) {  
				for (int i = 0; i < 100; i++) {  
					try {  
						Thread.class.wait();  
					} catch (InterruptedException e) {  
						throw new RuntimeException(e);  
					}  
					System.out.println("A");  
					Thread.class.notify();  
				}  
			}  
		}  
	};  
	Thread b = new Thread() {  
		public void run() {  
		synchronized (Thread.class) {  
			for (int i = 0; i < 100; i++) {  
				System.out.println("B"); 
				Thread.class.notify(); 
				try {  
					Thread.class.wait();  
				} catch (InterruptedException e) {  
					throw new RuntimeException(e);  
				}
			}  
		}  
	}; 
	a.start();  
	b.start();  
}  
  
public static void main(String[] args) {  
	LoginController loginController = new LoginController();  
	loginController.print();  
}
```

> [!note] *通过上面例子可以总结出一个规律，以前都是在线程对象之外使用锁，也就是在 run() 方法之外，在上面例子中在 run() 方法中加了锁，两者的区别是，加载线程对象之内，这个锁必须由这个线程获取，加载线程对象外，锁可以由任意对象获取。*

## 解决生产者、消费者问题

```java
static class MyBlockingQueue { 
	private Queue queue = null; 
	private int limit; 
	public MyBlockingQueue(int limit) { 
		this.limit = limit; 
		queue = new ArrayDeque<>(limit); 
	} 
	public synchronized void put(E e) throws InterruptedException { 
		while(queue.size() == limit) {  // 放之前如果队列满，则等待
			wait(); 
		} 
		queue.add(e); 
		notifyAll(); // 唤醒所有线程
	} 
	public synchronized E take() throws InterruptedException { 
		while(queue.isEmpty()) {  // 取之前如果为空，则等待
			wait(); 
		} 
		E e = queue.poll(); 
		notifyAll();  // 唤醒所有线程
	return e;  
	} 
}
```

_上述代码中，访问 put() 和 take() 的线程加入的是相同的等待队列。_
_所以需要使用 notifyAll 进行唤醒，如果使用 notify 唤醒的可能永远是同类线程。_

## 解决同时开始问题

```java
static class FireFlag { 
	private volatile boolean fired = false; 
	public synchronized void waitForFire() throws InterruptedException { 
		while(!fired) { 
			wait(); 
		} 
	} 
	public synchronized void fire() { 
		this.fired = true; 
		notifyAll(); 
	} 
}
```

_所有调用 waitForFire() 线程都将等待，当调用 fire（），所有等待中的线程都会唤醒，其他线程进入阻塞队列。因为 fired 变为 true，后面阻塞的队列获取到锁之后不会再进入等待队列，而是直接执行。_

## 解决等待结束问题

_使用 join(),join() 后面的代码将等待调用 join() 的对象结束后运行。_

_join 方法实际上调用的是 wait:_

```java
while (isAlive()) { 
	wait(0); 
}
```

_如果调用 join() 的线程是活的 isAlive() 将返回 true，join 一直等待。当线程结束后，Java 系统调用 notifyAll 来通知。_

## 解决异步调用问题

_FutureTask 类的 get 方法可以获得 Callable 接口的返回值。再获得返回值之前，也将阻塞主线程。_

## 解决集合点问题

## question

> [!question]+ *wait 和 notify 相关的方法为什么定义在 Object 类中，而不是 Thread 类中？*