---
aliases: 
tags: 
---

## 接口 Lock

*Lock 接口中提供了以下方法*

```plantuml
interface Lock{
{abstract} lock():void // 获取锁
{abstract} lockInterruptibly():void // 获取锁，可以响应中断
{abstract} tryLock():boolean // 尝试获取锁
{abstract} unlock():void // 释放锁
{abstract} newCondition():Condition // 新建一个条件
}
```

*可以看出，相比 synchronized，显式锁支持以非阻塞方式获取锁、可以响应中断、可以限时，这使得它灵活得多。*

*一般而言，显式锁的使用步骤如下：*

```java
Lock lock = new ReentrantLock();  // 声明锁
lock.lock(); //获取锁的过程不要写在 try 中，因为如果获取锁时发生了异常，异常抛出的同时也会导致锁释放 
try{ 
	。。。 // 临界区代码
}finally{ 
	lock.unlock(); //finally块中释放锁，目的是保证获取到锁之后最后一定能释放锁。 
}
```

> [!note] *显式锁需要显式声明后才能使用，隐式锁不需要显式声明，只需要使用就行，Java 中会自动加合适的锁。*

## 可重入锁 ReentrantLock

*可重入锁的特性和 synchronized 差不多：*
+ *可重入性：可以同时持有多个相同的锁。*
+ *可以解决竟态条件问题*
+ *可以保证内存可见性*

*下面通过一个例子来说明一下显式锁怎么用，例子来自《Java 编程的逻辑》551 页，*
*是一个银行账户之间相互转账的例子：*

```java
/**
* 表示银行账户的类
*/
public class Account { 
	private Lock lock = new ReentrantLock(); // 声明锁
	private volatile double money; 
	public Account(double initialMoney) { 
		this.money = initialMoney; 
	} 
	public void add(double money) { 
		lock.lock();  // 加锁
		try { 
			this.money += money; 
		} finally { 
			lock.unlock(); 
		} 
	} 
	public void reduce(double money) { 
		lock.lock(); // 加锁
		try { 
			this.money -= money; 
		} finally { 
			lock.unlock(); 
		} 
	} 
	public double getMoney() { 
		return money; 
	}
	void lock() { 
		lock.lock(); 
	} 
	void unlock() { 
		lock.unlock(); 
	} 
	boolean tryLock() { 
		return lock.tryLock(); 
	} 
}

public class AccountMgr { 
	public static class NoEnoughMoneyException extends Exception {} 
	/**
	* 转账
	*/
	public static void transfer(Account from, Account to, double money) throws
													NoEnoughMoneyException {
		from.lock(); 
		try { 
			to.lock(); 
			try { 
				if(from.getMoney() >= money) { 
					from.reduce(money); 
					to.add(money); 
				} else { 
					throw new NoEnoughMoneyException(); 
				} 
			} finally { 
				to.unlock(); 
			} 
		} finally { 
			from.unlock(); 
		}
	} 
}
```

*锁的访问规则就是：使用同一把锁的代码，在同一时间只能被一个线程方法。*
*在转账期间，也就是说 transfer() 方法以及 to 和 from 对象的 add() 和 reduce() 方法在同一个时间只能被一个线程访问，不然就会发生风险。*
*要想同时被一个时间访问这些代码需要使用同一个锁，但是这里显然没有使用同一个锁而是使用了两个锁，所以这里需要同时调用 from.lock() 和 to.lock(),这样在 from.lock() 和 to.lock() 以及 from.unlock() 和 to.unlock() 之间的代码以及其他被这两个锁修饰的地方也只能同时被一个线程访问。*

*to.lock() 到 to.unlock() 之间的代码同时加了两把锁，在一个线程同时获取到两把锁运行到这里之后，这两个所修饰的代码都不能被其他线程访问。*

### 使用 tryLock 避免死锁

*上面代码是会发生死锁的，因为 from.lock() 和 to.lock() 之间是有时间差的，如果有一个线程需要从 from 转钱到 to 账户，另一个线程需要从 to 转钱到 from 账户，第一个线程获取到了 from 的锁，然后去获取 to 的锁，第二个线程获取到了 to 的锁，然后去获取 from 的锁，这是就是会发生死锁。*

*tryLock 可以用来解决死锁：*

```java
public static boolean tryTransfer(Account from, Account to, double money) throws
														NoEnoughMoneyException { 
	if(from.tryLock()) { 
		try { 
			if(to.tryLock()) { 
				try { 
					if(from.getMoney() >= money) { 
						from.reduce(money); 
						to.add(money); 
					} else { 
						throw new NoEnoughMoneyException(); 
					} 
					return true; 
				} finally { 
					to.unlock(); 
				} 
			} 
		} finally { 
			from.unlock();
		} 
	} 
	return false; 
}
```

*tryLock() 用来尝试获取锁，如果获取成功则返回 true，如果失败则返回 false，无论如何都会返回并会像 lock() 一样如果失败就会阻塞线程。如果第二个锁获取失败则会执行下面代码，从而避免死锁：*

```java
 finally { 
	from.unlock();
} 
```

*一般而言 tryLock 的使用方法如下：*

```java
//实例化Lock接口对象
Lock lock = ...;
//根据尝试获取锁的值来判断具体执行的代码
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){
         
     }finally{
     	//当获取锁成功时最后一定要记住finally去关闭锁
         lock.unlock();   //释放锁
     } 
}else {
	//else时为未获取锁，则无需去关闭锁
    //如果不能获取锁，则直接做其他事情
}

```

