---
aliases: 
tags: 
---

> [!note] *ArrayList 的优化：指定初始容量，减少扩容次数。*

## 特点

+ _有序_
+ _允许 null 以及元素重复_
+ _线程不安全_
+ _底层为数组_

## 原理

*List 是 Java 中对线性表的实现，ArrayList 是线性表的顺序存储结构顺序表的实现。*

*与直接使用数组相比，线性表的优势就在于可以自动扩容、自动缩容。*

*ArrayList 的底层就是一个叫做 elementData 的 Object 数组，构造 ArrayList 的时候可以指定这个数组的初始容量，ArrayList 会使用指定的容量初始化 elementDate。*

> [!note] *也可以选择不指定初始容量，ArrayList 使用一个空的数组来初始化 elementData。*
> *不指定容量和指定容量为 0 这两种情况是不一样的。*
> *不指定容量使用 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 进行初始化，指定容量使用 EMPTY_ELEMENTDATA 进行初始化。*
> *虽然都是空数组，区别体现在扩容阶段。*

*ArrayList 的扩容发生在 add() 元素，并且 elementData 容量满的时候：*
+ *如果是第一次扩容，并且没有指定初始容量，则扩容至 10*
+ *其他情况下扩容至旧容量的 1.5 倍*

### 底层实现方式

_ArrayList 的底层为一个 Object 类型的数组：`elementData`。_

```java
transient Object[] elementData; //non-private to simplify nested class access
```

_ArrayList 有三个构造方法，**一个无参构造，两个有参构造**，构造方法的作用就是来初始化 elementData。_

_当使用无参构造来初始化 ArrayList 对象时，默认使用 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 数组来初始化 elementData 数组。`DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 数组是一个空数组，没有指定 size（即默认 size 为 0），第一次添加元素时才会扩容至 10。_

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
/**
 * 无参构造
 */
public ArrayList(){
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

_带容量的有参构造方法分为 **三种情况**：_
1. _==initialCapacity>0==：使用容量为 initialCapacity 的数组来初始化 elementData 数组_
2. _==initialCapacity\=\=0==：使用 EMPTY_ELEMENTDATA 数组来初始化 elementData 数组_
3. _==initialCapacity<0==: 抛异常_

```java
private static final Object[] EMPTY_ELEMENTDATA = {};
/**
 * 带容量的有参构造函数
 */
public ArrayList(int initialCapacity) {  
    if (initialCapacity > 0) {  
        this.elementData = new Object[initialCapacity];  
    } else if (initialCapacity == 0) {  
        this.elementData = EMPTY_ELEMENTDATA;  
    } else {  
        throw new IllegalArgumentException("Illegal Capacity: "+ 
                                           initialCapacity);  
    }  
}
```

> [!note] _为了防止频繁扩容带来的效率低下，在下面两种情况下应该指定容量进行初始化：_
> + _在知道会向 ArrayList 插入多少元素的情况下_
> + _在有大量数据写入时_

_还可以使用其他 Collection 集合来初始化 ArrayList 对象_

```java
/**  
 * 使用其他 Collection 类来初始化
 */
public ArrayList(Collection<? extends E> c) {  
    Object[] a = c.toArray();  
    if ((size = a.length) != 0) {  
        if (c.getClass() == ArrayList.class) {  
            elementData = a;  
        } else {  
            elementData = Arrays.copyOf(a, size, Object[].class);  
        }  
    } else {  
        // replace with empty array.  
        elementData = EMPTY_ELEMENTDATA;  
    }  
}
```

> [!tip]+ _**c.getClass()\=\=ArrayList.class 的作用**_
> _因为不同集合的 toArray() 方法的实现方式不同，如果 c 不是 ArrayList，则 toArray() 方法返回的的值不一定是 Object[],所以这里判断一下 c 是不是 ArrayList。_

### 扩容机制

_ArrayList 模式使用懒加载机制，在初始化时，并不分配内存，而是在第一次添加元素时才为数组分配内存。ArrayList 的默认容量为 10._

```java
private static final int DEFAULT_CAPACITY = 10;
```

> [!note] _使用无参构造进行初始化后的第一次 add() 时，扩容至默认容量 DEFAULT_CAPACITY = 10，其他情况扩容至原来容量的 1.5 倍。_

_ArrayList 中使用 size 变量来记录 ArrayList 的大小_

```java
private int size;
```

_java11 中将 add() 方法拆成了两个，保证字节码长度小于 35，JIT 的优化手段之一。_
_可以看到，调用 add() 方法添加元素时，并不是马上将元素添加到数组的尾部，而是首先会判断 elementData[] 的容量是不是满了，如果已经满了的话会调用 grow() 方法进行扩容。_

```java
private void add(E e, Object[] elementData, int s) {  
    if (s == elementData.length)  
        elementData = grow();   // 如果数组容量已满，则进行扩容
    elementData[s] = e;  
    size = s + 1;  
}  
  
public boolean add(E e) {  
    modCount++;  // 用于快速失败机制
    add(e, elementData, size);  
    return true;  
}
```

_grow() 方法要求扩容的大小为 1，也就是要求扩容的最小容量 minCapacity。_

```java
private Object[] grow() {  
    return grow(size + 1);  // 将容量扩容至size+1
}
private Object[] grow(int minCapacity) {  
// 使用Arrays.copyOf()方法将原数组拷贝到新开辟的数组
    return elementData = Arrays.copyOf(elementData,  
                                       newCapacity(minCapacity));  // 调用newCapacity()获取最             终的扩容大小
}
```

_最后实际上扩容的大小，grow() 方法会通过 newCapacity() 方法进行计算。
newCapacity() 方法返回的结果通过有两种情况：_
+ _使用无参构造后的第一次添加元素，返回容量为 10_
+ _其他情况返回旧容量的 1.5 倍_

```java
private int newCapacity(int minCapacity) {  
    // overflow-conscious code  
    int oldCapacity = elementData.length;// 目前的容量
    int newCapacity = oldCapacity + (oldCapacity >> 1); // 要扩容到的新容量，右移一位相当于除2，也就是说新容量为就容量的1.5倍  
    if (newCapacity - minCapacity <= 0) { 
	// 使用无参构造后的第一次添加元素时，计算出 oldCapacity 和 newCapacity都为0，此时会走这种情况
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)  
	    // 判断到底是不是使用无参构造后的第一次添加元素这种情况
            return Math.max(DEFAULT_CAPACITY, minCapacity); //   minCapacity为1，一次会返回DEFAULT_CAPACITY=10
        if (minCapacity < 0) // overflow  
        // 可能还会发生这种情况
            throw new OutOfMemoryError();  
        return minCapacity;  
    }  
    return (newCapacity - MAX_ARRAY_SIZE <= 0)  
    // 不是使用无参构造后的第一次添加元素的情况时，返回新容量，也就是就容量的1.5倍
    // MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
        ? newCapacity  
        : hugeCapacity(minCapacity);  
}
```
