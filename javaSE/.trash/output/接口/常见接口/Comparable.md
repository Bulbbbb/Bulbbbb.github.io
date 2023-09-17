---
aliases: 
tags: 
---
*Arrays.sort() 方法可以对对象数组进行排序，但是对象所属的类必须实现 Comparable 接口。*

*Comparable 接口中只有一个方法 compareTo()，所以任何实现了 Comparable 接口的类都会包含 compareTo() 方法。*

```java
public interface Comparable<T> {  
	public int compareTo(T o);  
}
```

*假如 X 是一个实现了 Comparable 接口的类，在调用 X.compareTo(Y) 来比较 X 和 Y 的时候：*
+ *当 X < Y 时：返回一个负数*
+ *当 X = Y 时：返回 0*
+ *当 X > Y 时：返回一个正数*

