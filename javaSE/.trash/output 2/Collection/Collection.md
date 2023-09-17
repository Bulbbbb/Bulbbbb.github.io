---
aliases: 
tags: 
---

```plantuml
@startuml
caption \n Collection接口框架图
interface Iterable
interface Collection
interface List
interface Set
interface Queue
abstract AbstractCollection
abstract AbstractList
abstract AbstractSet
abstract AbstractQueue
class ArrayList
class AbstractSequentialList
class LinkedList
class Vector
class HashSet
class TreeSet
Iterable <|-- Collection
Collection <|.. AbstractCollection
AbstractCollection <|-- AbstractList
List <|.. AbstractList
AbstractCollection <|-- AbstractSet
AbstractCollection <|-- AbstractQueue
Collection <|-- List
Collection <|-- Set
Collection <|-- Queue
List <|.. ArrayList
List <|.. LinkedList
List <|.. Vector
Set <|.. HashSet
Set <|.. TreeSet
AbstractList <|-- ArrayList
AbstractList <|-- AbstractSequentialList
AbstractSequentialList <|-- LinkedList
AbstractList <|-- Vector
AbstractSet <|-- HashSet
AbstractSet <|-- TreeSet
@enduml
```

> [!summary]+ 
_为了减轻开发者的工作，Java 为开发者提供了 AbstractCollection 等抽象类，以 AbstractCollection 抽象类为例，抽象类实现了 Collection 接口中的大部分方法，开发者在继承 Collection 接口后，只需要继承 AbstractCollection 抽象类就可以不用重新实现 Collection 接口的大部分方法。_
