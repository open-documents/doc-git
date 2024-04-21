
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
