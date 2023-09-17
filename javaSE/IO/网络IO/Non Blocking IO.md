---
aliases: 
tags: 
---
*Non Blocking IO：非阻塞 IO：*
*Non Blocking IO 解决的是 accept 阻塞和 read 阻塞的问题*
*下面就是非阻塞 IO 的具体逻辑，调用 accept 和 read 的时候，线程不阻塞，立即返回*
*与 BIO 相比，在没有 read 到数据的时候，可以马上去接受下一个请求*
*这里有一个小问题，客户端在连接之后，会马上接受数据，如果没有接收到则进入下一个请求，所以客户端根本来不及输入数据，然后就进入下一个循环 clientSocket 被刷新了。*
![[Pasted image 20230826135605.png]]


*解决上面问题的办法就是把 clientSocket 存起来，然后每次去轮询看有没有数据过来*
*所有的 client 都放到 list 里，然后每次有请求过来，都把 clientSocket 放到 list 里，然后去轮询 list 所有的 clientSocket，这样就可以解决 clientSocket 被覆盖的问题。*

> [!note] *通常我们会把数据处理的代码异步执行，轮询到某个 socket 有请求过来，就把 socket 放到别的线程执行，但是你不觉得光轮询的过程就占用太多时间了吗，如果有成千上万个 socket 怎么办？解决这个问题的办法就是 IO 多路复用*

![[Pasted image 20230826141026.png]]