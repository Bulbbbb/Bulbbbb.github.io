---
aliases: 
tags: 
---

## 特点

+ _无序_
+ _key 允许 null 值不允许重复（会覆盖），value 允许 null 值并且允许重复_
+ _线程不安全_
+ _底层为数组，链表，红黑树_

## 原理

*HashMap 为数据结构中哈希表的顺序存储结构的实现，底层为数组、链表和红黑树。*
*数组中存储的元素为 Node 类型，Node 是对 key-value 的抽象，在插入元素的时候，HashMap 通过 key 的 hashCode 来计算插入属于位置的下表，如果该位置处已经有元素，则在次下表处通过 Node 的 next 指针构建链表，通过链地址法解决冲突，在链表容量超过一定限度时将链表转化为红黑树。*

*HashMap 在构建的时候需要指定两个标准，一个是数组的初始容量，容量需要为 2 的 n 次方，一个是负载因子，HashMap 会在存储痛容量达到最大负载后进行扩容，负载因子默认为 0.75。*

*必须要为 2 的 n 次方是因为 hash 函数计算索引的时候利用的是位运算。*

*根据上面的需求需要注意的有以下几点*
*1.通过 HashCode() 计算插入下标*
*HashMap 中并不直接利用 Hash 值，而是将 hash 值的前 16 位与后 16 位进行异或，这样最后的结果就综合了 hash 值的前 16 位和后 16 位，可以减少碰撞的概率。*
*其次，hash 函数相当于取 hash 函数的前 x 位，改变容量可以改变下表*
*2 如果该位置处已经有元素，要判断插入的 元素是否和已存在的元素重合，如果重合则覆盖，否则才会添加*
*此时需要使用 equals 方法判断两个 key 是否相等，不需要使用 hashcode 进行判断，因为定位到同一下表处的两个 key 其 hashcode 一定相等*

*3 链表转化为红黑树*
*链表在容量大于 8 时妆化为红黑树，但不是直接转化为红黑树，如果数组的容量小于 64，会先通过扩容来解决冲突。*
*除此之外，如果红黑树容量小于 6 则退化为链表*

*4 扩容时机，在添加元素后如果容量超过负载也进行扩容*

### 底层实现

_HashMap 底层使用 Node<K,V> 类来抽象存储到 HashMap 中 key-value 对。_
_Node<K,V> 类继承自 Map.Entry<K,V> 类，我们将 key-value 对称为 Entry。_

```java
static class Node<K,V> implements Map.Entry<K,V> {  
    final int hash;  
    final K key;  
    V value;  
    Node<K,V> next;  
  
    Node(int hash, K key, V value, Node<K,V> next) {  
        this.hash = hash;  
        this.key = key;  
        this.value = value;  
        this.next = next;  
    }  
  
    public final K getKey()        { return key; }  
    public final V getValue()      { return value; }  
    public final String toString() { return key + "=" + value; }  
  
    public final int hashCode() {  
        return Objects.hashCode(key) ^ Objects.hashCode(value);  
    }  
  
    public final V setValue(V newValue) {  
        V oldValue = value;  
        value = newValue;  
        return oldValue;  
    }  
  
    public final boolean equals(Object o) {  
        if (o == this)  
            return true;  
        if (o instanceof Map.Entry) {  
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;  
            if (Objects.equals(key, e.getKey()) &&  
                Objects.equals(value, e.getValue()))  
                return true;  
        }  
        return false;  
    }  
}
```

_如果没有发生冲突，HashMap 默认使用一个 Node<K,V>数组来存储元素。_
_如果发生冲突，则使用 Node<K,V>中的 next 指针构造链表解决冲突。_

```java
transient Node<K,V>[] table;
```

![[Map.excalidraw.png|Map底层结构图]]

#### 初始化

_HashMap 提供了四个构造函数，一个无参，三个有参。_

> [!note] _需要注意的是，HashMap 默认是用懒加载机制，在初始化时并不分配内存，只进行相关参数的初始化。采用相同操作的还有 ArrayList。_

_初始化的参数主要为：_
+ _初始容量（capacity）：第一次要创建的桶的数量，也就是数组的容量，默认为 16。_
  _HashMap 中数组的容量必须是 2 的次幂，如果赋的初值不是 2 的次幂，则会通过 tableSizeFor() 函数调整为 2 的次幂。_
+ _负载因子（loadFactor）：允许桶的使用程度的最大负载。超过最大负载后扩容，默认为为 0.75。_

```java
// 构造函数一，无参
public HashMap() {  
    this.loadFactor = DEFAULT_LOAD_FACTOR;
}
// 构造函数二，指定容量.调用构造函数三
public HashMap(int initialCapacity) {  
    this(initialCapacity, DEFAULT_LOAD_FACTOR);  
}
// 构造函数三，指定初始容量和负载因子
public HashMap(int initialCapacity, float loadFactor) {  
    if (initialCapacity < 0)  
        throw new IllegalArgumentException("Illegal initial capacity: " +  
                                           initialCapacity);  
    if (initialCapacity > MAXIMUM_CAPACITY)  
        initialCapacity = MAXIMUM_CAPACITY;  
    if (loadFactor <= 0 || Float.isNaN(loadFactor))  
        throw new IllegalArgumentException("Illegal load factor: " +  
                                           loadFactor);  
    this.loadFactor = loadFactor;  
    this.threshold = tableSizeFor(initialCapacity);  
}
// 构造函数四，使用别的Map进行拷贝
public HashMap(Map<? extends K, ? extends V> m) {  
    this.loadFactor = DEFAULT_LOAD_FACTOR;  
    putMapEntries(m, false);  
}
```

### put

_`&` 的意思是按位与，在除数满足下面条件时可以使用 `&` 进行取余：_
+ _除数为 $2^n$_
_这就是为什么 HashMap 要求底层数组的长度为 $2^n$。_

_在计算机底层的运算逻辑中 $x/2^n==x>>n$。此时 x 保留下来的位为结果，而溢出区的位为余数。_
_所以假如要计算 $x\%b，（b=2^n）$，只要找出 x 在右移的过程中溢出的位就行。_
_在位运算中，取出一个二进制中的某位利用的就是与运算。也就是 `&`。_
_`&` 操作符为逻辑操作符 `与`，运算逻辑为：都为 1 则结果为 1，否则为 0。所以只要将要取出的那一位与 1 做与运算即可。比如 1011，要取第四位，只要 1011&1000 即可。_

_但是在利用 `&` 取余的过程，怎么知道要取哪几位呢。_
_可以被 2 整除的数有一个特别有趣的性质，1000=2^3,0111=2^3-1。也就是只要将 2^n 减 1，相当于对它的 2 进制形式按位取反。取反后原来的高位变为 0，低位变为 1，这样利用低位来取位，取出的数一定不大于 $2^n-1$,这也是为什么在利用 `&` 取余时必须为 2 的 n 次方。_
_而在我们计算余数的过程中，假如要计算 $x\%b$，（b=$2^n$），结果肯定是一个小于等于 b-1 的数。刚才我们说了，恰好 b-1 为一个低位全是 1 的数。所以说利用 b-1 来取出在取余过程中溢出的位恰好就是余数。_
_即 $x\%b$，$（b=2^n）==x \& (2^n-1)$_

```JAVA
if ((p = tab[i = (n - 1) & hash]) == null)  
    tab[i] = newNode(hash, key, value, null);
```

*这里 (n - 1) & hash 利用的就是这个原理，对 hash 进行取余，去一个不大于 n-1 的数。*n是桶个数。

#### put 的逻辑

1. _调用 Key 的 hashCode() 方法计算哈希值_

```java
public V put(K key, V value) {  
    return putVal(hash(key), key, value, false, true);  
}
static final int hash(Object key) {  
    int h;  
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  
}
```

2. _如果发现数组长度为 0，或者引用指向 null，则调用 resize() 进行扩容至 16_

```java
if ((tab = table) == null || (n = tab.length) == 0)  
    n = (tab = resize()).length;
```

3. _根据哈希值和数组长度来计算 index，如果没有发生哈希碰撞则直接新建 Entry，放入 tab[index] 处_

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,  
	。。。。。。
    if ((p = tab[i = (n - 1) & hash]) == null)  
        tab[i] = newNode(hash, key, value, null);  
	。。。。。。
}
```

4. _如果发生哈希冲突：_
   + _如果两个 Entry 的哈希值相等，并且使用=\=或者 equals() 判断的 key 相等,则直接使用新的结点进行覆盖。_
   + _如果 tab[index] 处是一个树，则调用 putTreeVal() 方法插入红黑树中。_
   + _如果 tab[index] 处是链表，则采用尾插法进行插入，_
      + _如果在遍历链表过程中发现 hash 值相等的元素并且使用=\=或者 equals() 判断的 key 也相等，则 break。_
      + _如果链表长度 7，则调用 treeifyBin() 转化为红黑树（尾插后进行判断，长度为 7，插入后为 8，也就是说链表长度为 8 时可能转化为红黑树）。_

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,  
               boolean evict) { 
    else {  
        Node<K,V> e; K k;  
        if (p.hash == hash &&  
            ((k = p.key) == key || (key != null && key.equals(k))))  
            e = p;  
        else if (p instanceof TreeNode)  
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  
        else {  
            for (int binCount = 0; ; ++binCount) {  
                if ((e = p.next) == null) {  
                    p.next = newNode(hash, key, value, null);  
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                        treeifyBin(tab, hash);  
                    break;  
                }  
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    break;  
                p = e;  
            }  
        }  
        if (e != null) { // existing mapping for key  
            V oldValue = e.value;  
            if (!onlyIfAbsent || oldValue == null)  
                e.value = value;  
            afterNodeAccess(e);  
            return oldValue;  
        }  
    }  
}
```

#### hash 函数

_HashMap 中利用下面公式求元素的索引：_

$$index = (n - 1) \& hash$$

_其中 n 为数组的容量，hash 为 hash() 函数获取的 hash 值。_
_上面我们说过，2^x 这样的数很特别。对于 2^x，其第 x+1 位为 1，从 1 到 x 位为 0。_
_如果将其减 1，即 2^x-1，则情况正好相反，其第 x+1 位为 0，从 1 到 x 位为 1,相当于取反。_
_上面函数中，假设 n=2^x，则 n-1 从 1 到 x 位为 1，与 hash 进行&操作，相当于取 hash 的第 1 到 x 位。这里也能看出来，改变容量可以改变 index。_
==_综上，HashMap 中的 Entry 的 index 就是 hash 的前 x 位。_==

_hash 是通过 hash() 函数获取的，hash() 函数会调用 key 的 hashCode() 函数获取 hashCode，但是并不直接利用获取的 hashCode。_

```java
static final int hash(Object key) {  
	int h;  
	return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  
}
```

_可以看到，hash() 函数中通过调用 hashCode() 函数获取到了 h，然后将 h 与 (h >>> 16) 进行异或操作。_
_hashCode() 函数返回的 h 是一个 32 位的 int 类型的整数，右移 16 位相当于将高 16 位移到低 16 位。也就是说 hash 最后的取值综合了 h 的高 16 位和低 16 位。原因是因为如果只考虑低位会加大碰撞的概率。_

### 扩容

#### 初始化 table（懒加载）

_与 ArrayList 相同，HashMap 默认使用懒加载机制，当调用无参构造时，HashMap 的默认容量为 16，但是在初始化时，并不分配内存。而是在第一次 put 时分配内存。_
_初始容量必须是 2 的 n 次幂。_

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

_put() 方法的底层为 putVal() 方法，putVal() 方法会判断是不是第一次 put。_

```java
if ((tab = table) == null || (n = tab.length) == 0)  
    n = (tab = resize()).length;
```

_如果是第一次 put 则调用 resize() 进行扩容。_
_resize() 中也有专门针对是不是第一次的判断。如果是则使用 DEFAULT_INITIAL_CAPACITY=16 进行默认初始化。_

```java
else {               // zero initial threshold signifies using defaults  
    newCap = DEFAULT_INITIAL_CAPACITY;  // 使用默认初始容量进行初始化
    newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);  
}
threshold = newThr;  
@SuppressWarnings({"rawtypes","unchecked"})  
Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
```

#### size 超过 threshold 后扩容

_当 HashMap 中 Entry 个数超过 threshold，HashMap 会进行扩容。_
_threshold = capacity（容量）\*loadFactor（负载因子）。_

_扩容后的容量=原容量<<1,也就是原容量的 2 倍。_

#### resize() 方法

_HashMap 中调用 resize() 进行扩容，扩容是需要将 Entry 进行重新分配位置，这个时候要考虑下面 3 中情况：_
1. _数组的 index 位置处为单个元素_
2. _数组的 index 位置处为链表_
   _如果 index 处为链表，则旧链表会被拆成两个新链表 lo 和 hi，然后遍历旧链表处的元素，如果 (e.hash & oldCap) \=\= 0,则将该节点存放到 lo 链表中，否则存放到 hi 链表中。_
   _这里分析一下 (e.hash & oldCap) \=\= 0 这个条件，首先我们要知道：_
   1. _旧链表长度为 2 的次幂，假设为 $2^m$_
   2. _新链表的长度为旧链表长度的 2 倍，即 $2^{m+1}$_
   3. _HashMap 计算 index 的方法为 (n - 1) & hash_
	_假如，旧链表长度为 16，index 为 hash 的低 4 位，假设为 abcd，新链表长度为 32，index 为 hash 的低 5 位，这 5 位数有两种情况。_
    + _0abcd_
    + _1abcd_
	_`0abcd`=`abcd`，index 不变。而 `1abcd` = `0abcd + 10000` = `0abcd + oldCap`。_
	_可以看到，index 是否变化就体现在第 4 位上，而 (e.hash & oldCap) 正好可以取到这一位。_
	_所以如果 (e.hash & oldCap) \=\= 0,则将该节点存放到 lo 链表中，否则存放到 hi 链表中_
	_假设拆分成链表的 index 位 j，lo 链表的 index 不变也为 j，hi 链表的位置在 j+oldCap。_
3. _数组的 index 位置处为红黑树_
	_如果 index 处位红黑树，则会调用 split() 函数将旧红黑树拆分长两棵小红黑树 lo，hi。_
	_当 lo 或者 hi 中的节点数小于 6 时会转变成链表。_

### 数组中元素转化为链表

_使用 hash 值 (hashCode() 方法) 和容量计算出 index 后，如果该 index 处已经有元素了，证明发生 hash 碰撞。_
_hash 碰撞后判断是否重复，即调用两个 Entry 的 key 对象的 equals() 方法判断是否重复。_
_如果重复，则覆盖。_
_如果不重复则插入链表或者红黑树。_

> [!note] _先调用 hashCode（）方法，再调用 equals() 方法。hashCode() 相等的元素在同一个 index 处，但是 hashCode() 相等的元素 equals() 不一定相等。_

_也就是说当 equals() 相等时，数组中某一个 index 处会转变成链表。_

### 链表转化为红黑树

_当链表的长度大于 8 时，会发生下面两种情况：_
+ _数组长度小于 64，通过扩容改变次 Entry 的 index。_
+ _数组长度不小于 64，将链表转化为红黑树。_

### 红黑树的退化

_当红黑树中元素个数小于 6 时，红黑树将退化为链表。这里不使用 8 是防止反复震荡。_

_HashMap 会在下面两种情况下判断是否将红黑树转化为链表：_
1. _调用 resize() 进行扩容时 --》[[#resize() 方法]]_
2. _调用 remove() 移除元素时_
   _当删除元素时，会检查红黑树是否满足退化条件，此时的退化与结点的数目无关。_
   + _根结点为空_
   + _根的左子树为空_
   + _根的右子树为空_
   + _跟的左子树的左子树为空_
