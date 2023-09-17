---
aliases: 
tags: 
---

## IntegerCache

_IntegerCache 是 Integer 的内部类，该类维护一个 Integer 数组，在初始化时，通过静态代码块对 Integer 数组进行初始化，创建 [-128,127] 范围内的 Integer 对象并将其引用存储到 Integer 数组中。_

_在创建 Integer 对象时，会判断该对象的值是否在 [-128,127] 范围内，如若在，则直接返回已经创建好的 Integer 对象的引用，如果不在则创建对象。_

_所以说，值在 [-128,127] 范围内，且值相同的 Integer 对象的引用都是唯一的，指向同个 Integer 对象。_

## Integer 中的 equals

 _使用 Integer 中的 equals() 方法比较两个 Integer 对象时，会先把对象拆箱成基本数据类型然后进行比较。_

## question

> [!question]- _Integer a= 127 与 Integer b = 127 相等吗？_
> _在通过=\=判断两个对象是否相等时，判断的时其指向的内存地址是否相等_
> _值在 [-128,127] 范围内，且值相同的 Integer 对象的引用都是唯一的，指向同个 Integer 对象，所以 a =\= b_
