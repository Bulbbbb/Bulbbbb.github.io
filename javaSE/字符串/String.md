---
aliases: 
tags: 
---
*String 是 Java 中最经常用来表示字符串的类：*
*需要注意的是，通过字面量赋值和使用 new 赋值是不同的。*
![[Pasted image 20230625153856.png]]

```java
String s1="sssws";

String s2=new String("sssss");
```

*String 底层默认使用 char 数组实现：*
![[Pasted image 20230625143259.png]]
*char 数组使用 final 修饰，也就是不可修改的。*
*这里需要注意的是，不可修改的是 vaule 的指向，value 中的内容是可以修改的。*
*也就是说 string 内部维持一个不可变的引用，这个引用始终指向同一个地址。并且 String 内部并没有提供任何可以改变此地址内容的方法，所以就是不可变的。*

```java
String s1=new String("sssws");
s1=new String("aaa") ;
```

*上面的操作中，并不是改变 s1 内部 char 数组的指向，而是创建了一个新数组，s1 指向了这个新数组。*

*value 指向的地方叫做 String 常量池，在创建 String 之前，java 首先在 String 常量池中寻找有没有创建过相同的 char 数组，如果已经创建过，则令 String 的 value 指向这个数组，否则在 String 常量池中创建新数组。*

*所以 String s1=new String("sssws"); 实际上会创建两个对象，一个常量池中的 char 数组对象，一个堆区中的 String 对象。*


*需要注意：String a="sss"; 之创建一个对象，不会再堆上创建对象。*

### intern()

*intern() 可以创建 String，方法负责从常量池中寻找是否存在相同的 char 数组，如果存在则直接返回，常量池中的地址。相当于通过字面量创建。*
