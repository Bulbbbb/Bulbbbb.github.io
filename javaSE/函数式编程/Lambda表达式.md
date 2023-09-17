---
aliases: 
tags: 
---
*Lambda 表达式用来传递一个代码片段。*
*在 Lambda 表达式出现之前，由于 Java 是一个面向对象的编程语言，要想传递代码片段，必须将代码片段包装成一个对象。*

*接口常用来传递代码片段，例如 Arrays.sort() 可以传递一个 Comparator 对象。这里需要的并不是这个对象而是对象中的 compare() 方法。*

```java
int compare(T o1, T o2);
```

*我们可以通过匿名内部类来简写：*

```java
Arrays.sort(integers, new Comparator<Integer>() {  
	@Override  
	public int compare(Integer o1, Integer o2) {  
		return 0;  
	}  
});
```

## Lambda 表达式

*在 Java8 中提供了一种新的传递代码的语法 Lambda 表达式。*
*上述方法可以利用 Lambda 表达式简写为：*

```java
Arrays.sort(integers,（Integer o1, Integer o2）—>{
	return 0; 
});
```

> [!note] *匿名内部类和 lambda 表达式都是用来创建对象。*

*匿名内部类通过两种方式来创建对象：*
+ *继承一个类（可以是抽象的可以是具体的）*
+ *实现接口*
*lambda 只能通过实现接口来创建对象。*
*且此接口必须为函数式接口。*

*函数式接口就是只有一个抽象方法的接口。*