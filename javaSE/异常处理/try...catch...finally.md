---
aliases: 
tags: 
---
*try...catch...finally 用来捕捉异常并且处理。*
1. *不管 try 是否捕捉到异常，finally 都会运行*
2. *try 没有捕捉到异常，catch 不会运行*
3. *当 try 和 catch 中有 return 时，finally 仍然会执行；*
4. *finall 在 return 之后执行，return 结果会保存到栈中，如果后续有 return 语句会覆盖*