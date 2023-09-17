---
aliases: 
tags: 
---

## ConcurrentHashMap 的线程安全问题

*ConcurrentHashMap 的线程安全问题主要为两方面：*
+ *JDK7,死循环、数据丢失，发生在扩容过程中*
+ *JDK8，数据覆盖，发生在 put 的过程中*
*JDK7 中 ConcurrentHashMap 采用分段锁来保证线程安全问题*

*JKD7 中 ConcurrentHashMap 的问题主要出现在扩容后的转移函数 transfer() 中:*

```java
void transfer(Entry[] newTable, boolean rehash) {         
	int newCapacity = newTable.length;     // 获取桶个数    
	for (Entry<K,V> e : table) {    // 遍历桶         
		while(null != e) {                 
			Entry<K,V> next = e.next;   // 使用 next 指针指向下一个结点              
			if (rehash) { // 是否需要重新计算hash值                    
				e.hash = null == e.key ? 0 : hash(e.key);                 
			}                 
			int i = indexFor(e.hash, newCapacity);  // 计算索引               
			/* 使用头插法插入 */
			e.next = newTable[i];             
			newTable[i] = e;                 
			e = next;             
		}        
	}     
}
```

*假设有两个线程同时对下面 HashMap 进行扩容：*
![[Pasted image 20230510161059.png|未扩容前的HashMap|300]]
*两个线程都会新建一个 HashMap，扩容 2 倍：*
![[Pasted image 20230510161352.png|300]]
*线程 2 在插入 a 时执行到下面代码时由于时间片耗尽被挂起：*

```java
Entry<K,V> next = e.next;
```

> [!note] *此时线程 1 和线程 2 的状态都是一样的，e 指向 a，next 指向 b。*

*此时：e=a,next=b，线程 1 继续执行，*
![[Pasted image 20230510161744.png|300]]
*在线程 1 将结点转移完成后，线程 1 被挂起，线程 2 继续执行：*
![[Pasted image 20230510161916.png|300]]
*线程 2 停止时，e=a,next=b，继续执行下面代码：*
*执行完之后就变为下面这种情况，由于 a.next 本来就为 null，所以这里没断链：*
![[Pasted image 20230510163026.png|250]]
*继续执行，由于 next 为 b，所以 e=next 操作后 e 也指向 b，e 不为 null，继续执行 while 循环：*
*此时 next 指向 e.next，所以 next 又指向了 a，newTable[i] 指向 a，所以 e.next 即 b 指向了 a，newTable[i] 指向 e 即 b，随后 e 指向 a：*
![[Pasted image 20230510165048.png|250]]

*继续执行，next 指向 e.next,即 null，e.next 指向 newTable[i]，即 b，newTable[i] 指向 e，即 a，e 指向 next，即 null。*
![[Pasted image 20230510165319.png|250]]
*此后因为 e 为 null，所以执行完毕。可以看到内部形成了一个循环*


*如果此时线程 2 在未将 newTable 设置给 table 之前进行读会发生死循环，造成 CPU100%。*
*如果线程 2 将 newTable 设置给 table ，结点 c 会丢失*
*这就是 JDK7 导致的死循环和数据丢失问题。*

*此问题在 JDK8 中已经改善，JDK8 在 resize() 进行上面操作，并且采用尾插法。*
*但是 JDK8 中同样会带来覆盖的问题。*

## ConcurrentHashMap

*底层数据结构：数组、链表、红黑树。*
*JDK8 中通过使用了 `Node` + `CAS` + `Synchronized` 来保证线程安全。*
*这里通过 put() 来说明一下 ConcurrentHashMap 是如何保证线程安全的。*

## table 初始化时的线程安全

*和 HashMap 相同，ConcurrentHashMap 在第一次 put() 时对 table 进行初始化，因为可能有多个线程同时 put(),所以可能会发生多次初始化，所以 ConcurrentHashMap 需要保证 table 只初始化一次。*

```java
private transient volatile int sizeCtl; // 默认为0
private final Node<K,V>[] initTable() {  
	Node<K,V>[] tab; int sc;  
	// 自旋
	while ((tab = table) == null || tab.length == 0) {  // 只有table为空时才能进入循环内
		if ((sc = sizeCtl) < 0)  // sizeCtl = -1，说明有线程在初始化table
			Thread.yield(); // lost initialization race; just spin  ,让出时间片
		else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {  // 通过CAS将 sizeCtl 设置为 -1
			try {  
				// 第二次检查是否队列为空，
				if ((tab = table) == null || tab.length == 0) {  
					int n = (sc > 0) ? sc : DEFAULT_CAPACITY;  // 默认初始容量为16，sc>0,说明初始化时传值了
					@SuppressWarnings("unchecked")  
					Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];  // 创建table，容量为n
					table = tab = nt;  // 将table和tab指向 nt
					sc = n - (n >>> 2);   // 下次扩容的临界值
				}  
			} finally {  
				sizeCtl = sc;  
			} 
			break;  // 初始化成功后跳出循环
		}  
	}  
	return tab;  
}
```

*可以看到，initTable() 是通过 CAS 和 sizeCtl 来控制 table 的初始化只进行一次的。*
*初始化的过程中进行了双重检测来判断 table 是否为空，原因看这里 -->:[java.util.concurrent.ConcurrentHashMap#initTable方法中双重检测是否有必要](https://blog.csdn.net/JoshuaYss/article/details/114043006)*

## put 时的线程安全

```java
public V put(K key, V value) {  
	return putVal(key, value, false);  
}
final V putVal(K key, V value, boolean onlyIfAbsent) {  
	if (key == null || value == null) throw new NullPointerException();  
	int hash = spread(key.hashCode());  
	int binCount = 0;  
	for (Node<K,V>[] tab = table;;) {  // CAS 机制
		Node<K,V> f; int n, i, fh;  
		if (tab == null || (n = tab.length) == 0)  // 此一次入队时扩容
			tab = initTable();  
		else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) { // 查看索引处是否已有元素
			/*没有发生hash碰撞，通过CAS将Node插入table[i]处*/
			if (casTabAt(tab, i, null,new Node<K,V>(hash, key, value, null)))  
				break; // no lock when adding to empty bin  
		}  
		else if ((fh = f.hash) == MOVED)  
			/*发生hash碰撞，此处节点为转移结点，协助扩容*/
			tab = helpTransfer(tab, f);  
		else {  
			/*发生hash碰撞*/
			V oldVal = null;  
			/*进入下面代码的线程需要首先获得头节点的锁*/
			synchronized (f) {  // 加锁，f 是tabAt的返回值，这里是链表的头节点
			
				/*确保一下put操作不会受到扩容的影响，头节点还是那一个头节点
				* 如果头节点换了，可能至少有两个桶的操作使用一个锁，影响并发性能
				*/
				if (tabAt(tab, i) == f) {  
					if (fh >= 0) {  // hash值大于等于0，
						binCount = 1;  // 长度
						for (Node<K,V> e = f;; ++binCount) {  // 死循环，直到将结点添加到队尾
							K ek; V ev;
							/*hash值相等，进行覆盖*/
							if (e.hash == hash &&  
								((ek = e.key) == key || (ek != null && key.equals(ek)))) {  
								oldVal = e.val;  
								if (!onlyIfAbsent)  
									e.val = value;  
								break;  
							}  
							/*hash值不相同添加到队尾*/
							Node<K,V> pred = e;  
							if ((e = e.next) == null) {  
								pred.next = new Node<K,V>(hash, key,value, null);  
								break;  
							}  
						}  
					}  
					/*红黑树*/
					else if (f instanceof TreeBin) {  
						Node<K,V> p;  
						binCount = 2;  
						if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,value)) != null) {  
																		oldVal = p.val;  
							if (!onlyIfAbsent)  
								p.val = value;  
						}  
					}  
				}  
			}  
			/*判断是否转化为红黑树*/
			if (binCount != 0) {  
				if (binCount >= TREEIFY_THRESHOLD)  
					treeifyBin(tab, i);  
				if (oldVal != null)  
					return oldVal;  
				break;  
			}  
		}  
	}  
	addCount(1L, binCount);  // 判断是否进行扩容
	return null;  
}
```

### 没有发生 hash 碰撞

*CAS 操作 保证了 放入数据的时候 当前节点一定是 null 的 如果 在这期间 有别的线程也来操作这个卡槽的值，那 CAS 中 table 中 i 位置的值 就一定不是 null 了，和预期值不符，那就 CAS 执行失败，进入下次循环 下次循环的时候 发现此槽位已经有值了 就会走到 Hash 碰撞后的 Synchronized 的方法块里面，这样保证了 在新的卡槽 新增第一个元素的是线程安全的*

### 发生 hash 碰撞

*ConcurrentHashMap 在插入节点时会对结点进行加锁，锁是加在头节点上面的，可以细化锁粒度，多个线程可以同时对不同的桶进行操作。*

## 扩容时的线程安全

*[(74条消息) ConcurrentHashMap1.8 - 扩容详解_ZOKEKAI的博客-CSDN博客](https://blog.csdn.net/ZOKEKAI/article/details/90051567)*

```java
private final void addCount(long x, int check) {  
	CounterCell[] as; long b, s;  
	/*计算键值对个数*/
	if ((as = counterCells) != null ||  
	!U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {  
		CounterCell a; long v; int m;  
		boolean uncontended = true;  
		if (as == null || (m = as.length - 1) < 0 ||  
		(a = as[ThreadLocalRandom.getProbe() & m]) == null ||  
		!(uncontended =  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {  
			fullAddCount(x, uncontended);  
			return;  
		}  
		if (check <= 1)  
			return;  
		s = sumCount();  
	}  
	/*检查是否需要扩容*/
	if (check >= 0) {  
		Node<K,V>[] tab, nt; int n, sc;  
		//当上面计算出来的值对个数超出sizeCtl时，触发扩容，调用核心方法transfer
		while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&  
										(n = tab.length) < MAXIMUM_CAPACITY) {  
			int rs = resizeStamp(n);  
			if (sc < 0) {  
				if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||  
				sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||  
				transferIndex <= 0)  
					break;  				
				// 如果并发扩容，transfer直接使用正在扩容的新hash表，保证了不会出现hash表覆盖的情况
				if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))  
					transfer(tab, nt);  
			}  
			else if (U.compareAndSwapInt(this, SIZECTL, sc,  (rs << RESIZE_STAMP_SHIFT) + 2))  
				transfer(tab, null);  
			s = sumCount();  
		}  
	}  
}
```