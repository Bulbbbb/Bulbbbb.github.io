---
aliases: 
tags: 
---
_Map 用于保存具有映射关系的数据，因此 Map 里保存着两组值，一组值用于保存 Map 里的 key，另一组值用于保存 Map 里的 value。_
![[Pasted image 20230320102359.png|分开看Map的key组和value组]]

## Map 接口相关操作

```plantuml
@startuml
caption \n Map接口提供的方法
interface Map{
..增加..
{abstract} put()
{abstract} putAll()
..删除..
{abstract} remove()
{abstract} clear()
..修改..
{abstract} replace()
{abstract} compute()
..查询..
{abstract} get()
{abstract} keySet()
{abstract} values()
{abstract} size()
..判断..
{abstract} isEmpty()
{abstract} containsKey()
{abstract} containsValue()
}
@enduml
```

> [!tip]+ _compute() 是一个使用频率非常高的方法。_

## Map 接口的实现

```plantuml
@startuml
caption \nMap接口框架图
interface Map
abstract AbstractMap
abstract Dictionary
class HashMap
class TreeMap
class HashTable
class LinkedHashMap
class Properties
Map <|.. HashMap
Map <|.. TreeMap
Map <|.. HashTable
Map <|.. AbstractMap
AbstractMap <|-- HashMap
AbstractMap <|-- TreeMap
Dictionary <|-- HashTable
HashMap <|-- LinkedHashMap
HashTable <|-- Properties
@enduml
```

> [!info] _HashMap 继承自 Abstract 抽象类。_

## question

> [!question]- _说说 HashMap 和 HashTable 的区别_
