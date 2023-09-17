---
aliases: 
tags: 
---

## Collection 接口中的抽象方法

![](attachment/b40857aa7f0031709ef20a3555a5d950.png)

> [!note]
> _集合中不能存放基本数据类型，使用 add 方法增加基本数据类型时，会进行自动装箱。_

## AbstractCollection 抽象类

![](attachment/b39814bac26b7368381ff459a17137ec.png)

_`AbstractCollection类 ` 在实现 `Collection接口 ` 时实现了除 `iterator()` 和 `size()` 外的所有方法，因此 `AbstractCollection 类` 要求子类必须实现上述上述两个方法。_

_除此之外，`AbstractCollection类 ` 虽然实现了 `add(E e) 方法 `，但在方法体中却直接抛出了异常：`throw new UnsupportedOperationException()`，因此默认是不支持添加单个元素的，其子类要想支持添加单个元素则需要重写此方法。_

```java
public boolean add(E e) {
    throw new UnsupportedOperationException();
}
```

> [!question] _**既然 AbstractCollection 类中没有实现 `add(E e)` 的逻辑，为什么不将它定义成抽象方法？**_
>
> _对于这个问题，如果将 `add(E e)` 方法定义成抽象方法，那么不管子类支不支持单个元素插入都要重写 `add(E e)`，而使用抛异常操作代替抽象方法的话，如果子类不支持单个元素插入就可以不用重写此方法。相当于减少了工作量。_

_其次，java 的设计者们可能想规范这种行为，如果让开发者们自己去实现的话，可能有各种各样的行为来提示其他开发者不支持这个方法。_

## question

> [!question]- _Collection 里有哪些常用方法？_
