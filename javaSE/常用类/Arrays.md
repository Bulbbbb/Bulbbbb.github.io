---
aliases: 
tags: 
---

## Arrays 类中的静态方法

```plantuml
@startuml
caption \n Arrays类中的静态方法
class Arrays{
{static}binarySearch( )
|||
{static}copyOf( )
|||
{static}copyOfRange( )
|||
{static}equals( )
|||
{static}fill( )
|||
{static}sort( )
|||
{static}toString( )
|||
}
note right of Arrays::"binarySearch( )"
二分查找
end note
note right of Arrays::"copyOf( )"
复制一个新数组
end note
note right of Arrays::"copyOfRange( )"
复制部分元素
end note
note right of Arrays::"fill( )"
填充
end note
@enduml
```

###   Java8 中为 Arrays 类新增的方法

_Java8 中为 Arrays 类新增了许多方法，这些方法可以利用 CPU 的并行能力来增强性能。_

```plantuml
@startuml
caption \nJava8 中为 Arrays 类新增的方法
class Arrays{
|||
{static} parallelPrefix（ ）
|||
{static} setAll（ ）
|||
{static} parallelsetAll（ ）
|||
{static} parallelSort（ ）
|||
{static} spliterator（ ）
|||
{static} stream（ ）
|||
}
note right of Arrays::"parallelPrefix（ ）"
使用op参数指定的计算公式得到的计算结果作为新的元素
end note
note right of Arrays::"setAll（ ）"
使用生成器（generator）为数组元素设置值
end note
note right of Arrays::"parallelSort（ ）"
增加并行能力的排序
end note
note right of Arrays::"spliterator（ ）"
将数组转换为Spliterator对象
end note
note right of Arrays::"stream（ ）"
将数组转换为Stream
end note
@enduml
```

### 方法使用实例

```java
public class Demo {  
    public static void main(String[] args) {  
        int[] arr1 = new int[]{3, -4, 25, 16, 30, 18};  
        Arrays.parallelSort(arr1); // 对数组进行排序  
        System.out.println(Arrays.toString(arr1)); // [-4, 3, 16, 18, 25, 30]  
  
        int[] arr2 = new int[]{3, -4, 25, 16, 30, 18};  
        Arrays.parallelPrefix(arr2, new IntBinaryOperator() {  
            @Override  
            public int applyAsInt(int left, int right) {  
//                left代表数组中前一个元素，计算第一个元素时，left为1  
//                right代表数组中当前索引处的元素  
                return left * right;  
            }  
        });  
        System.out.println(Arrays.toString(arr2)); // [3, -12, -300, -4800, -144000, -2592000]  
  
        int[] arr3 = new int[5];  
        Arrays.parallelSetAll(arr3, new IntUnaryOperator() {  
  
            @Override  
            public int applyAsInt(int operand) {  
//                operand代表正在计算的元素索引  
                return operand * 5;  
            }  
        });  
        System.out.println(Arrays.toString(arr3)); // [0, 5, 10, 15, 20]  
    }  
}
```
