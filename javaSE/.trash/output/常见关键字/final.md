---
aliases: 
tags: 
---
_final 可用于修饰类、变量和方法，用于标识类、方法和变量不可改变。_

*final 修饰类，不能被继承*
*final 修饰变量，不能被改变*
*final 修饰方法，不能被重写*
*被 final 修饰的方法，jvm 会尝试内联*

## final 修饰成员变量

_final 修饰的成员变量必须由程序员显示地指定初始值。_
+ _类变量：必须在类变量声明时，或者在静态代码块中指定初始值。_
+ _成员变量：必须在非静态代码块，或者声明该变量时，或者在构造器中指定初始值。_
成员变量不能在静态代码快中初始化，因为静态代码快不能访问非静态变量
## final 修饰局部变量

_final 修饰局部变量时可以不指定初始值，可以在后面的代码中赋初始值。_
+ _如果在声明式指定初始值则后面代码中不能再赋初始值。_
+ _如果声明式没有指定初始值可以再后面代码中赋初始值，但是只能赋一次。_

## 可执行 " 宏替换 " 的 final 变量

_对一个变量来说，不管是类变量，实例变量还是局部变量，只要满足 3 个条件就可以视为一个直接量：_
+ _使用 final 修饰符修饰_
+ _在定义该 final 变量时指定了初始值（可以使用表达式）_
+ _该初始值可以再编译时确定下来_

_这样的变量被称为“宏变量”，编译器会把宏变量直接替换为宏变量的值。_

*也就是编译期常量*

> [!note] _只有在定义是赋初值才是宏变量，再构造器中和静态代码块中赋初值不是宏变量。_

## final 修饰基本数据类型和引用数据类型的区别

+ _修饰基本数据类型时，不可以二次初始化_
+ _修饰引用数据类型时，不可以改变引用的指向，但可以改变指向变量的内容。_

## final 方法

_final 修饰的方法不可以被重写，例如 Object.getClass() 方法。_

## final 类

_final 修饰的类不可以有子类。_

### 不可变类

_不可变类是指其实例再创建后不能更改的类。_
_在 Java 中，8 中包装类以及 String 都是不可变类。_

_如何需要创建不可变类，可尊循下面原则：_
+ _使用 private 和 final 修饰符来修饰该类的成员变量_
+ _提供带参数构造器，用于传入参数来初始化类里的成员变量_
+ _仅为该类的成员变量提供 getter 方法，不要为该类的成员变量提供 setter 方法，因为普通方法无法修改 final 修饰的成员变量。_
+ _如果有必要，重写 Object 类的 hashCode() 和 equals() 方法。equals() 方法根据关键成员变量来作为两个对象是否相等的标准，除此之外，还应该保证两个用 equals() 方法判断为相等的对象的 hashCode() 也相等。_

> [!note] _这里使用 final 修饰看似没有任何意义，因为在 private 可以保证类外无法修改变量的值，而不提供 setter() 方法可以保证类内的方法不会修改变量的值。
> 如果能保证这两点确实不需要 final 修饰，但是，在编程的时候可能会无意识的修改变量的值，从而造成错误。而 final 可以避免这种错误的产生。
> final 实现不可变类是一种最简单的方式，同时，使用 private final 修饰可以增加可读性，快速确立该类为不可变类。_

> [!note] *只提供 getter 方法，将变量设置为 private 和 final，使用构造器来初始化。*

## question

> [!question]- _final 有什么用？_
> _final 可以用来修饰类、属性和方法：_
> + _被 final 修饰的类不可以被继承_
> + _被 final 修饰的方法不可以被重写_
> + _被 final 修饰的变量不可以被改变（被 final 修饰不可变的是变量的引用，而不是引用指向的内容，引用指向的内容是可以改变的）_

> [!question]- _抽象类能使用 final 修饰吗？_
> _不能，定义抽象类就是让其它类继承的，如果定义为 final 该类就不能被继承，这样就是产生矛盾，所以 final 不能修饰抽象类。_

> [!question]- _final、finally、finalize 区别？_
> *finalize 在对象被垃圾回收之前调用。*
> *finally 用于捕捉异常*
> *final 定义常量*
