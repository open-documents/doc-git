



Excutor是Excutor框架最基本的接口，只有一个excute()方法。

Excutor使得任务和执行任务的容器分离开来。

Excutor的实现类也会实现ExecutorService接口，ExecutorService接口增强了Excutor接口的功能。

Excutors提供了Excutor实现类的工厂方法来创建各种Excutor的实现类。

Excutor的实现类：
- ForkJoinPool
- ThreadPoolExecutor
- PrivilegedExecutor（jdk.internal.net.http）


ExcutorService是Excutor接口的扩展接口，增加了对执行器生命周期的管理。

执行器的终止状态：1）不允许新任务提交；2）没有正在进行的任务；3）没有等待开始的任务。



下面是ExcutorService的定义：
```java
public interface ExecutorService extends Executor {
	void shutdown();
	List<Runnable> shutdownNow();
	boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
	
	<T> Future<T> submit(Callable<T> task);
	<T> Future<T> submit(Runnable task, T result);
	Future<?> submit(Runnable task);
	
	<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;
    
    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;

	boolean isShutdown();
	boolean isTerminated();
}
```

-- --
方法一共分为下面几类：

# 1）关闭

一个没有使用的执行器应该被关闭以释放资源。

关闭方法：
shutdown()：正在执行的任务和任务队列中的任务会继续执行，只是不再接收新的任务。
shutdownNow()：
- 尝试暂停所有正在执行的任务，让所有任务队列中的任务不会再执行并且以List的形式返回任务队列中的所有任务。
- 这个方法不会等待正在执行的任务执行完成
awaitTermination()：一直阻塞，直到在一个关闭命令执行后还会执行的任务全部执行完毕。

一般的关闭流程：
```java
void shutdownAndAwaitTermination(ExecutorService pool) {
	pool.shutdown(); // 不再接收新任务
    try {
	    // 等待正在执行中的任务和任务队列中的任务执行完毕
	    if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
	        pool.shutdownNow(); // 尝试暂停正在执行的任务，暂停所有任务队列中等待执行的任务
	        // 因为正在执行的任务完全终止需要时间,因此等待正在执行的任务完全终止
	        if (!pool.awaitTermination(60, TimeUnit.SECONDS))
	            System.err.println("Pool did not terminate");
	    }
    } catch (InterruptedException ie) {
        // (Re-)Cancel if current thread also interrupted
        pool.shutdownNow();
        // Preserve interrupt status
        Thread.currentThread().interrupt();
    }
}
```

# 2）submit

submit()是对excute()的增强，返回Future类型结果，



ThreadPoolExecutor的整体运行逻辑如下：
- 随着新任务的到来，如果已有核心线程数小于设置的核心线程数，那么会创建核心线程来处理这个新任务，即使有核心线程空闲也还是会创建核心线程来处理新任务。
- 当核心线程数达到最大后，新任务会被加入到队列中。
- 如果核心线程数达到最大以及任务队列满了后，会创建非核心线程去处理新任务。
- 如果核心线程和非核心线程都处在运行状态，并且任务队列满了，后续的任务会被拒绝处理。



# Executors提供的实例对象

ThreadPoolExecutor的实例一般都是通过Executors提供的工厂函数来创建。
Executors提供下面几种特殊的ThreadPoolExecutor实例创建：


## SingleThreadExecutor

SingleThreadExecutor顾名思义，只有一个线程的线程池。

只有一个线程的线程池的执行器的特点：
- 只有一个核心线程，保证任务线性完成。
- 阻塞队列大小无限。
- 与newFixedThreadPool(1)不同，singleThreadExecutor不能重新配置来添加额外的核心线程。
- 核心线程在执行器关闭之前如果在执行期间因为错误而终止，如果后续还有任务需要执行，执行器会创建一个新的核心线程来代替它。

下面是Executors提供的两个工厂方法，用来创建只有一个线程的线程池ThreadPoolExecutor实例。
```java
public static ExecutorService newSingleThreadExecutor() {  
    return new FinalizableDelegatedExecutorService  
        (new ThreadPoolExecutor(1, 1,  
                                0L, TimeUnit.MILLISECONDS,  
                                new LinkedBlockingQueue<Runnable>()));  
}
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {  
    return new FinalizableDelegatedExecutorService  
        (new ThreadPoolExecutor(1, 1,  
                                0L, TimeUnit.MILLISECONDS,  
                                new LinkedBlockingQueue<Runnable>(),  
                                threadFactory));  
}
```

## FixedThreadPool

FixedThreadPool顾名思义，线程池的大小是固定。

线程池大小固定ThreadPoolExecutor的特点：
- 线程池大小固定，阻塞队列大小无限。
- 核心线程会随着新任务的到来而一一创建。
- 当全部核心线程处于使用状态时，新任务会在阻塞队列中排队。
- 核心线程在执行器关闭之前如果在执行期间因为错误而终止，如果后续还有任务需要执行，执行器会创建一个新的核心线程来代替它。


下面是Executors提供的两个工厂方法，用来创建线程池大小固定的ThreadPoolExecutor实例。
```java
public static ExecutorService newFixedThreadPool(int nThreads) {  
    return new ThreadPoolExecutor(nThreads, nThreads,  
                                  0L, TimeUnit.MILLISECONDS,  
                                  new LinkedBlockingQueue<Runnable>());  
}

public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {  
    return new ThreadPoolExecutor(nThreads, nThreads,  
                                  0L, TimeUnit.MILLISECONDS,  
                                  new LinkedBlockingQueue<Runnable>(),  
                                  threadFactory);  
}
```

## CachedThreadPool

CachedThreadPool顾名思义，里面的线程都是缓存起来的，超过一段时间（默认60s）没有使用会被回收。
CachedThreadPool没有核心线程，最大线程数是Integer.MAX_VALUE。

CachedThreadPool按需增加线程，擅长处理大量耗时短的异步任务。

下面是Executors提供的两个工厂方法，用来创建CachedThreadPool实例。
```java
public static ExecutorService newCachedThreadPool() {  
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,  
                                  60L, TimeUnit.SECONDS,  
                                  new SynchronousQueue<Runnable>());  
}
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {  
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,  
                                  60L, TimeUnit.SECONDS,  
                                  new SynchronousQueue<Runnable>(),  
                                  threadFactory);  
}
```






# 构造函数

构造函数必须传递的参数：
- 核心线程数。
- 最大线程数。
- 最大存活时间和时间单位。
- 阻塞队列。

构造函数可选参数：
- 线程工厂。
- 拒绝执行处理器。

ForkJoinPool不是用来替代ThreadPoolExecutor，而是对实际线程池使用场景的一种补充。

ForkJoinPool与ThreadPoolExecutor不同在于：
- ThreadPoolExecutor只维护一个任务队列，所有线程从该任务队列中取任务执行。
- ForkJoinPool中的每个线程（ForkJoinWorkerThread）都维护一个任务队列（ForkJoinPool.WorkQueue）。

# 介绍

首先，关于ForkJoinTask的介绍，看  。
-- --
接下来介绍一下ForkJoinPool.WorkQueue。

WorkQueue是一个任务队列，每个线程（ForkJoinWorkerThread）维护一个这样的任务队列。
WorkQueue内部通过维护一个ForkJoinTask数组来表示任务队列。
WorkQueue内部维护一个指向ForkJoinPool和ForkJoinWorkerThread（代表这个WorkQueue属于哪个ForkJoinWorkerThread）的指针。

-- --
接下来介绍一下ForkJoinWorkerThread。

ForkJoinWorkerThread是Thread的一个简单子类，没有重写Thread中的方法。
ForkJoinWorkerThread内部维护了一个任务队列

-- -- 
最后介绍一下主角ForkJoinPool。

ForkJoinPool中执行任务的线程不是普通线程，而是ForkJoinWorkerThread。

ForkJoinPool内部维护的是一组WorkQueue，而不是一组ForkJoinWorkerThread。

# 使用



`invoke(ForkJoinTask<T>)`

`execute(ForkJoinTask<?>)`
`execute(Runnable)`

`submit(ForkJoinTask<T>`
`submit(Callable<T> task)`
`submit(Runnable task, T result)`

