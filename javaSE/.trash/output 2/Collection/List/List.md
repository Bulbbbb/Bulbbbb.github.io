---
aliases: 
tags: 
---

## List 接口相关操作

_List 接口继承自 Collection 接口，相比于 Collection 接口，List 接口增加了许多方法，这些方法基本上与 index 相关。_

```plantuml
@startuml
caption \n List接口中新增的方法
interface List{
..增加..
{abstract} add ( index, E )
{abstract} addAll ( index, c )
..删除..
{abstract} remove ( index )
..修改..
{abstract} set ( index, E )
{abstract} replaceAll ( )
..查询..
{abstract} get ( index )
{abstract} indexOf ( O )
{abstract} lastIndexOf ( O )
{abstract} listIterator ( )
{abstract} subList( fromIndex, toIndex ）
..排序..
{abstract} sort ( )
}
@enduml
```

^9ea51a

## List 接口的实现

```plantuml
@startuml
caption \n List接口的实现
interface Iterable
interface Collection
interface List
abstract AbstractList
abstract AbstractSequentialList
class ArrayList
class LinkedList
class Vector
Iterable <|-- Collection
Collection <|-- List
List <|.. AbstractList
List <|.. ArrayList
AbstractList <|-- ArrayList
AbstractList <|-- AbstractSequentialList
List <|.. LinkedList
AbstractSequentialList <|-- LinkedList
List <|.. Vector
AbstractList <|-- Vector
@enduml
```

> [!question] _**AbstractList 抽象类为什么要继承 AbstracCollection 抽象类？**_
>
> _AbstractList 抽象类应该实现 List 接口中的大部分方法，其中大部分方法已经在 AbstractCollection 抽象类中实现了，所以就可以通过继承 AbstractCollection 抽象类而不必实现这些方法。_
