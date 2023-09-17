---
aliases: 
tags: 
---
_synchronized 关键字可以解决竟态条件和内存可见性问题。_

*synchronized，Java 中的互斥锁，通过保证线程互斥来解决竟态条件问题。*
*synchronized 可以修实例方法、静态方法、代码块。*

## 用法

+ _修饰实例方法_
+ _修饰静态方法_
+ _修饰代码块_

### synchronized 实例方法

_对该实例方法所属的实例加锁，该实例内 synchronized 方法同一时间内只能被一个线程访问。_

> [!note] _需要注意，synchronized 保护的是对象而不是代码，如果该对象的有两个 synchronized 方法 a()、b()。如果有一个线程在访问 a(),那 同一时间内 b() 方法也是不能被其他线程访问的。只能访问非 synchronized 方法。_

_可以将 同一个对象内的 synchronized 方法当作一个整体。这个整体在同一时间内只能被一个线程访问。_


_被保护的对象有一个锁和一个等待队列，锁只能被一个线程持有，其他试图获取同样锁的线程需要等待，加入等待队列，线程的状态被阻塞。_

```java
public class Counter {  
	private int count;  
	  
	public synchronized void setCount() {  
		count++;  
	}  
	  
	public synchronized int getCount() {  
		return count;  
	}  
}
class test extends Thread {  
	Counter counter;  
	  
	public test(Counter counter) {  
		this.counter = counter;  
	}  
  
	@Override  
	public void run() {  
		for (int i = 0; i < 100; i++) {  
			counter.setCount();  
		}  
}  
  
public static void main(String[] args) throws InterruptedException {  
	Counter counter1 = new Counter();  
	test[] tests = new test[100];  
	for (int i = 0; i < 100; i++) {   // 创建100个线程
		tests[i] = new test(counter1);  
		tests[i].start();  
	}  
	for (int i = 0; i < 100; i++) {  
		tests[i].join();  
	}  
	System.out.println(counter1.getCount());  
	  
	}  
}
```

_在上面例子中，创建了一个线程类 test 类，接受一个 Counter 类型的变量 counter1。test 类的方法体中调用 Counter 类中的 setCount() 方法每次对 counter 变量 +1。_
_main() 方法中创建了 100 个线程同时运行。_

_如果没有使用 synchronized 关键字修饰，最终打印出来的 count 并不能保证是 1000。_

> [!note] _上面的例子有 3 个注意事项:_
> 1. _**线程数一定要多**_
>    _这里我们使用了 100 个线程，如果线程数太少可能不会同时调用修改 count。_
> 2. _**在调用 getCount() 方法之前使用 join()**_
>    _当在线程 B 中执行 ThreadA.join 时，线程 B 会被阻塞，等到线程 A 运行完成。_
>    _如果不使用 join() 方法，getCount() 方法可能会在 test 线程之前调用，就不能看到最后的结果，结果可能为 0。_
> 3. _**synchronized 关键词要不要同时修饰 setCount() 方法和 getCount() 方法**_
>    _其实只需要修饰 setCount() 方法就可以保证最后结果的准确性，如果在线程体中调用了 getCount() 方法，那我们也不能确定 getCount() 方法和 setCount() 方法的调用顺序。_

### synchronized 静态方法

_synchronized 静态方法保护的类对象。_

> [!note] _注意区分类对象和类的对象，类的对象通过 new 关键字获取，类的对象通过下面方法获取：_
> + _类型.class：    String.class_
> + _对象.getClass()：    "hello".getClass()_
> + _Class.forName()：   Class.forname("java.lang.String")_

_我们将上面例子中 Counter 类中的变量和方法使用 static 修饰，线程类中的代码不变：_

```java
public class Counter {  
	private static int count;  
	  
	public synchronized static void setCount() {  
		count++;  
	}  
	  
	public synchronized static int getCount() {  
		return count;  
	}  
}
```

_最后的结果并不能保证为 10000。_
_也就是通过实例对象可以同时访问 setCount() 方法和 getCount() 方法，因为访问的是实例对象的方法，而不是访问的 class 对象的方法。_

### synchronized 代码块

_使用 synchronized 修饰的代码块同样是将锁加在对象上，该对象使用 synchronized 修饰的代码块同一时间只能一个线程访问。_

```java
/**
* 等价 synchronized 实例方法
*/
public class Counter {  
	private static int count;  
	  
	public static void setCount() {  
		synchronized(this){
			count++;  
		}
	}  
	  
	public static int getCount() {  
		return count;  
	}  
}
/**
* 等价 synchronized 静态方法
*/
public class Counter {  
	private static int count;  
	  
	public static void setCount() {  
		synchronized(StaticCounter.class){
			count++;  
		}
	}  
	  
	public static int getCount() {  
		return count;  
	}  
}
```

## synchronized 的特性

### 可重入性

_可重入性说的是，对同一个线程，他在获得了锁之后，在调用其他需要同样锁的代码时，可以直接调用。_

```java
public class Counter {  
	private static int count;  
	  
	public synchronized void setCount() {  
		count++;  
		System.out.println(this.getCount());  
	  
	}  
	  
	public synchronized int getCount() {  
		return count;  
	}  
}
```

_上面代码中，线程访问 setCount() 时需要申请锁，在持有锁的同时去访问另一个需要锁的方法 getCount() 可以直接调用。_

*两个锁必须是同一个锁。*

> [!note] _简单的理解就是在一个 synchronized 修饰的区域区域调用另一个 synchronized 修饰的代码区域。注意：并不是在一个线程体内顺序调用两个 synchronized 修饰的代码区域。_

_可重入性通过记录锁的持有线程和持有数量来实现。_
_在一个被 synchronized 保护的代码被调用时，先检查是否加锁：_
+ _未加锁_
	+ _锁被当前线程持有 --> 增加持有数量_
	+ _锁不是被当前线程持有 --> 线程加入等待队列_
+ _未加锁 -->当前线程调用_

_只有当持有数量为 0 时，才释放整个锁。_

### 保证内存可见性

```java
public class Switcher { 
	private boolean on; 
	public boolean isOn() 
	{ 
		return on; 
	} 
	public void setOn(boolean on) 
	{ 
		this.on = on; 
	} 
}
```

_上面代码中对 on 变量的操作很明显为原子操作，不会发生竟态条件。还需要加 synchronized 吗？_

_需要，synchronized 除了保证原子操作外，它还有一个重要的作用，就是保证内存可见性，在释放锁时，所有写入都会写回内存，而获得锁后，都会从内存中读最新数据。_

> [!note] *导致内存可见性的原因有两个：*
> + *修改后不会立即刷新到内存中*
> + *读取变量的时候不会到内存中读而是在缓存中读*

_不过，如果只是为了保证内存可见性，使用 synchronized 的成本有点高，有一个更轻量级的方式，那就是给变量加修饰符 ==volatile==，如下所示:_

```java
public class Switcher { 
	private volatile boolean on; 
	public boolean isOn() 
	{ 
		return on; 
	} 
	public void setOn(boolean on) 
	{ 
		this.on = on; 
	} 
}
```

### 死锁

_使用 synchronized 或者其他锁，要注意死锁。所谓死锁就是类似这种现象，比如，有 a、b 两个线程，a 持有锁 A，在等待锁 B，而 b 持有锁 B，在等待锁 A，a 和 b 陷入了互相等待，最后谁都执行不下去。_

_所以要注意，尽量避免再持有一个锁的同时去申请另一个锁。_
_也就是不要在 synchronized 修饰的代码内去申请另一个 synchronized 修饰的代码，应该按照顺序申请。_


_同一个锁指的是同一个对象的锁。_
