---
aliases: 
tags: 
---
*JDBC,Java 数据库连接。*

*在 Java 中使用 JDBC 有 6 大步骤：*
+ *注册驱动*
![[Pasted image 20230613183803.png]]
+ *获取连接*
![[Pasted image 20230613183829.png]]
+ *获取 statement*
![[Pasted image 20230613183914.png]]
+ *执行 sql*
![[Pasted image 20230613183936.png]]
+ *处理结果集*
![[Pasted image 20230613184002.png]]
+ *释放连接*
	*注意连接关闭的顺序与连接关闭的正确姿势*
	*如果没有使用线程池，可以只关闭 connection，如果使用了线程池，必须全部关闭。*
![[Pasted image 20230613184251.png]]
