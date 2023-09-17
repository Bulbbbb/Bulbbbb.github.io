---
aliases: 
tags: 
---
*Cloneable 是一个标记接口，接口中没有任何的方法：*

```java
public interface Cloneable {  
}
```

*标记接口的唯一作用是在类型检查中使用 instanceof：*

```java
 if (obj instanceof Cloneable) ...
```

## 浅拷贝

*在 Java 中可以使用赋值运算符‘=’实现对象的拷贝：*

```java
var original = new Employee( "John Public" ,50000);
Employee copy = original;
```

*在这种拷贝机制之下，原对象及其副本都是同一个对象的引用，即他们指向相同的对象，任何一个变量的改变都会影响另一个变量。我们将这种拷贝机制成为==浅拷贝==。*

## 深拷贝（克隆）

*现在我们希望副本 copy 是一个新对象，并且 copy 的初始状态与 original 相同，二者的修改互不影响，这种拷贝机制我们成为==深拷贝==，或者叫做==克隆==。这种形况下，我们需要借用 Object.clone() 实现。*

```java
Employee copy = original.clone(); // 调用 Object.clone() 
copy.raiseSalary(10);
```

*为了安全考虑，Object 类将 clone() 方法定义为 protect。即子类只能调用自己的 clone() 方法。*

*Object.clone() 是一个 native 方法，虽然使用的是深拷贝但是还是会造成错误。*
*比如下面的复制情况，虽然调用 clone() 深拷贝了一个对象，copy 指向这个对象。但是*
*对象内部的值是原样不动复制过来的，相当于浅拷贝。*

![[Pasted image 20230513214814.png|400]]

*如果复制的对象中都是基本变量或者 final 常量，不可变类是没有任何问题的。但是大多数情况下不是这样的，所以 Object 类中将 clone() 方法定义为 protected，只能在自己的类中复制自己，因为你很清楚复制的内容是什么。*
*而要想调用其他对象的 clone() 方法，那你就必须重写该对象的 clone() 方法，将其访问权限修改为 public，再重写过程中思考要怎么拷贝。*

> [!note] *按照上面的意思，可以记忆为：*
> *浅拷贝：直接通过 = 实现*
> *深拷贝：新开辟一块内存*
> *半深不浅拷贝：对象是深拷贝，但对象中组合的其他用的是浅拷贝*

### 重写 clone()

*如果默认的 clone 方法不满足要求或者该对象需要克隆，就需要重写 clone() 方法。需要实现下面两部操作：*
1. *==实现 Cloneable 接口==。*
2. *重写 clone() 方法，将访问修饰符修改为 public。*

```java
class Employee implements Cloneable{
	. . .
	public Employee clone( ) throws CloneNotSupportedException{
	
		Employee cloned = (Employee) super.clone(); // 使用默认的拷贝方法拷贝Employee
		cloned.hireDay = (Date) hireDay.clone() ; // 将Employee内需要深拷贝的对象进行深拷贝
		return cloned;
	}
	. . .
}
```
