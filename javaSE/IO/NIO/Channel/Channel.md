---
aliases: 
tags: 
---

## Channel

```plantuml
interface Channel{
	{abstract} isOpen()
	{abstract} close()
}
```

*数据总是需要在两个数据源之间进行读写操作，这个动作由谁完成，放在具体的数据源中并不现实，没法做到代码复用。*
*在 Java 中，将这些动作拆分出来，由一个第三者完成，这个第三者叫做通道（Channel）或者流（Stream）。*

### Channel 的分类

```plantuml
@startuml

class AbstractInterruptibleChannel
class AbstractSelectableChannel
interface ByteChannel << interface >>
interface Channel << interface >>
class DatagramChannel
class FileChannel
interface GatheringByteChannel << interface >>
interface InterruptibleChannel << interface >>
interface MulticastChannel << interface >>
interface NetworkChannel << interface >>
interface ReadableByteChannel << interface >>
interface ScatteringByteChannel << interface >>
class SelectableChannel
class ServerSocketChannel
class SocketChannel
interface WritableByteChannel << interface >>

AbstractInterruptibleChannel  -[#008200,dashed]-^  Channel                      
AbstractInterruptibleChannel  -[#008200,dashed]-^  InterruptibleChannel         
AbstractSelectableChannel     -[#000082,plain]-^  SelectableChannel            
ByteChannel                   -[#008200,plain]-^  ReadableByteChannel          
ByteChannel                   -[#008200,plain]-^  WritableByteChannel          
DatagramChannel               -[#000082,plain]-^  AbstractSelectableChannel    
DatagramChannel               -[#008200,dashed]-^  ByteChannel                  
DatagramChannel               -[#008200,dashed]-^  GatheringByteChannel         
DatagramChannel               -[#008200,dashed]-^  MulticastChannel             
DatagramChannel               -[#008200,dashed]-^  ScatteringByteChannel        
FileChannel                   -[#000082,plain]-^  AbstractInterruptibleChannel 
FileChannel                   -[#008200,dashed]-^  ByteChannel                  
FileChannel                   -[#008200,dashed]-^  GatheringByteChannel         
FileChannel                   -[#008200,dashed]-^  ScatteringByteChannel        
GatheringByteChannel          -[#008200,plain]-^  WritableByteChannel          
InterruptibleChannel          -[#008200,plain]-^  Channel                      
MulticastChannel              -[#008200,plain]-^  NetworkChannel               
NetworkChannel                -[#008200,plain]-^  Channel                      
ReadableByteChannel           -[#008200,plain]-^  Channel                      
ScatteringByteChannel         -[#008200,plain]-^  ReadableByteChannel          
SelectableChannel             -[#000082,plain]-^  AbstractInterruptibleChannel 
SelectableChannel             -[#008200,dashed]-^  Channel                      
ServerSocketChannel           -[#000082,plain]-^  AbstractSelectableChannel    
ServerSocketChannel           -[#008200,dashed]-^  NetworkChannel               
SocketChannel                 -[#000082,plain]-^  AbstractSelectableChannel    
SocketChannel                 -[#008200,dashed]-^  ByteChannel                  
SocketChannel                 -[#008200,dashed]-^  GatheringByteChannel         
SocketChannel                 -[#008200,dashed]-^  NetworkChannel               
SocketChannel                 -[#008200,dashed]-^  ScatteringByteChannel        
WritableByteChannel           -[#008200,plain]-^  Channel                      
@enduml

```

*Java 中常用的 Channel 的 channel 有 4 中类型：*
+ *FileChannel*
+ *SocketChannel*
+ *ServerSocketChannel*
+ *DatagramChannel*

*Channel 只能与缓冲区交互，与 Stream 相同，Channel 也是有方向的：*
+ *从文件流向缓冲区：read*
+ *从缓冲区流向文件：write*

### 获取 Channel

*方法一：调用 getChannel() 方法，Java 中可以获取 Channel 的对象如下：*
+ *FileInputStream*
+ *FileOutputStream*
+ *RandomAccessFile*
+ *DatagramSocket*
+ *Socket*
+ *ServerSocket*

*方法二：通过 Files.newByteChannel() 获取字节通道*
*方法三：通过通道的静态方法 open() 打开并返回指定通道*

*对文件进行读写必须获取文件描述符。*
