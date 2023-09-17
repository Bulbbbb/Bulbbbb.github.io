---
aliases: 
tags: 
---
*在通过 Thread 实现线程的时候，我们发现任务和任务的执行是绑定在一起的，可以理解为每一个线程都绑定了一个任务。有多少个任务就必须创建多少个线程来执行。*
*Executor 是一个异步任务执行框架，其基本思想就是将任务和任务的执行分离。*
*通过将任务和任务的执行分离，可以分类来管理任务和任务的执行，可以在不同的线程上执行不同的任务。*

```java
public interface Executor {  
	void execute(Runnable command);  
}
```

*可以看到 Executor 中只有一个方法 execute() ,接收 Runnable 类型的参数。*
*因此，可以将 Executor 看作任务执行的抽象，Runnable 看作是任务的抽象。*


*ExecutorService 是 Executor 的子接口，提供了更多的服务。*

*ExecutorServicez 提供了 submit() 来表示任务的提交，和 execute() 是一样的效果：*

```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```

*可以看到，扩展后的 Executor 不仅支持 Runnable，也支持了 Callable。*

> [!note] *需要注意的是任务提交后并不一定立即执行。*

*submit() 的返回值为 Future，Future 用来表示异步执行的结果。*

> [!note] *submit() 用来提交 callable，execute() 用来执行 Runnable。*

*通过 Future 可以查询当前任务的执行状态，取消任务，获取任务的返回结果。*

```java
public interface Future<V> {  
	boolean cancel(boolean mayInterruptIfRunning);  
	boolean isCancelled();  
	boolean isDone();  
	V get() throws InterruptedException, ExecutionException;  
	V get(long timeout, TimeUnit unit)  
					throws InterruptedException, ExecutionException, TimeoutException;  
}
```

*get() 方法用来获取任务的执行结果，任务的结果大概有以下几种：*
1. *正常完成，get 方法会返回其执行结果，如果任务是 Runnable 且没有提供结果，返回 null。*
2. *任务执行抛出了异常，get 方法会将异常包装为 ExecutionException 重新抛出，通过异常的 getCause 方法可以获取原异常。*
3. *任务被取消了，get 方法会抛出异常 CancellationException。*
4. *如果调用 get 方法的线程破中断了，get() 方法会抛出 InterruptedException。*

*结果有两个：返回结果、返回异常*

> [!note] *Future 的存在为任务的执行与任务的提交提供了纽带。*

*现在我们可以来总结以下异步执行的基本框架：*
1. *实现了任务和任务执行的分离*
2. *使用 Runnable 和 Callable 表示要执行的任务*
3. *使用 Executor 和 ExecutorService 表示执行器*
4. *使用 Future 表示任务的执行结果*

