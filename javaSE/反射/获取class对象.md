---
aliases: 
tags: 
---
*获取一个类 class 对象有下面几种方法：*
+ Class.forName()

```java
Class Dog {  }
Class c1 = Class.forName("Dog");
```

+ *对象的 getClass() 方法*

```java
Dog d1 = new Dog()
Class c1 = d1.getClass();
```

+ *.class*

```java
Class c1 = Dog.class;
```

+ *基本数据类型*

```java
Class<Integer> type = Integer.TYPE;
```

> [!note] *对象的 class 属性和 forName() 方法，对象的 getClass() 方法。*