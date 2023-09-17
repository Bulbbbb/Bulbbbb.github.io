---
aliases: 
tags: 
---
*内部同样通过 char 数组实现。*
![[Pasted image 20230625150147.png]]
不同的是 char 数组没有使用 final 修饰，指向可变。
*在重新赋值的时候实际上改变的是 value 的指向。*

*value 同样指向常量池。*


*字符串通过“+”运算符拼接的操作默认通过 StringBuilder 实现：*

```java
String str1 = new String("111111") ;
String str2 = new String("222222");
String str = str1 + str2;
System.out.println(str);
```

*相当于：*

```java
String str1 = "111111";
String str2 = "222222";
StringBuilder sb = new StringBuilder();
sb.append(str1);
sb.append(str2);
String str = sb.toString();
System.out.println(str);
```

*分析一下，上面操作中创建了 7 个对象。*