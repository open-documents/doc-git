
	1.当线程池小于corePoolSize时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程。 
	2.当线程池达到corePoolSize时，新提交任务将被放入workQueue中，等待线程池中任务调度执行 
	3.当workQueue已满，且maximumPoolSize>corePoolSize时，新提交任务会创建新线程执行任务 
	4.当提交任务数超过maximumPoolSize时，新提交任务由RejectedExecutionHandler处理 
	5.当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程 
	6.当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭 

需要掌握：

）

）对于corePoolSize和maximumPoolSize的设置线程不能和execute线程并发。因为maximumPoolSize在两个方法中不提供同步。



Excutors提供的池：

创建一个线程池，该线程池根据需要创建新线程，但在以前构建的线程可用时将重用它们。这些池通常会提高执行许多短期异步任务的程序的性能。要执行的调用将重用以前构建的线程（如果可用）。如果没有可用的现有线程，将创建一个新线程并将其添加到池中。60秒未使用的线程将终止并从缓存中删除。因此，闲置足够长时间的池不会消耗任何资源。请注意，可以使用ThreadPoolExecutor构造函数创建具有相似属性但细节不同（例如，超时参数）的池。

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



创建一个线程池，该线程池重用在共享无边界队列中运行的固定数量的线程。在任何时候，最多nThreads线程将是活动的处理任务。如果在所有线程都处于活动状态时提交了其他任务，它们将在队列中等待，直到有一个线程可用。如果任何线程在关闭之前的执行过程中由于失败而终止，则在需要执行后续任务时，将替换一个新的线程。池中的线程将一直存在，直到它被显式关闭


创建一个执行器，该执行器使用单个工作线程在无边界队列中操作。（但是，请注意，如果在关闭之前执行过程中由于失败而导致此单个线程终止，则如果需要执行后续任务，将替换一个新的线程。）任务保证按顺序执行，并且在任何给定时间都不会有多个任务处于活动状态。与其他等效的newFixedThreadPool（1）不同，返回的执行器保证不可重新配置以使用额外的线程。







# 属性

## 构造属性

```java
private volatile int corePoolSize;                  // 存在get和复杂set方法
private volatile int maximumPoolSize;               // 存在get和复杂set方法
private volatile long keepAliveTime;
private final BlockingQueue<Runnable> workQueue;

private volatile ThreadFactory threadFactory;       // 存在get和set方法
private volatile RejectedExecutionHandler handler;  // 存在get和set方法

//get方法:allowsCoreThreadTimeOut()
//set方法:allowCoreThreadTimeOut(boolean value)
private volatile boolean allowCoreThreadTimeOut;

private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();


```

corePoolSize

public void setCorePoolSize(int corePoolSize)

设置线程的核心数。这将重写构造函数中设置的任何值。如果新值小于当前值，多余的现有线程将在下次空闲时终止。如果更大，则将在需要时启动新线程以执行任何排队任务。

maximumPoolSize

public void setMaximumPoolSize(int maximumPoolSize)

设置允许的最大线程数。这将重写构造函数中设置的任何值。如果新值小于当前值，多余的现有线程将在下次空闲时终止。

keepAliveTime

public void setKeepAliveTime(long time, TimeUnit unit)

如果设置的值比以前小，那么在方法返回前，终止所有空闲时间比新值大的线程（核心线程根据属性来看）。
如果设置的值比以前大，不做处理。


## 功能属性

**核心属性**：`private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));`

前3位记录该ThreadPoolExcutor状态：

- RUNNING。
- SHUTDOWN：不接收新任务，但是处理队列中的任务。
- STOP：既不接受新任务，也不处理队列中的任务，同时还要中断正在运行的任务。
- TIDYING：所有的任务都已经停止，而且workCount值为0，同时还会运行`terminate()` hook方法。
- TERMINATED：`terminate()`方法执行完成。



其他静态常量定义如下：
```java
private static final int COUNT_MASK = (1 << COUNT_BITS) - 1;  // COUNT_BITS = 29
// runState is stored in the high-order bits  
private static final int RUNNING    = -1 << COUNT_BITS;  
private static final int SHUTDOWN   =  0 << COUNT_BITS;  
private static final int STOP       =  1 << COUNT_BITS;  
private static final int TIDYING    =  2 << COUNT_BITS;  
private static final int TERMINATED =  3 << COUNT_BITS;
```
具体值如下：
![[Pasted image 20221215222402.png]]











特点：

**1）按需施工（On-demand construction)**

默认情况，即使是 *核心线程* 也只会在新任务到来时*被创建(be created)* 和 *被运行(be started)*，但是这个默认行为可以通过方法 `prestartCoreThread` 和 `prestartAllCoreThreads` 来改变。

在用一个非空队列来构建线程池时可能有这样的需求。






# 属性方法

```java
private static int runStateOf(int c)     { return c & ~COUNT_MASK; }  // 取高3位，返回32位
private static int workerCountOf(int c)  { return c & COUNT_MASK; }   // 取低29位，返回32位
private static int ctlOf(int rs, int wc) { return rs | wc; }          // 拼接32位

private static boolean runStateLessThan(int c, int s) {  return c < s;}  
private static boolean runStateAtLeast (int c, int s) {  return c >= s;}  

private static boolean isRunning(int c) {  return c < SHUTDOWN;}
```

## setCorePoolSize(int corePoolSize) : void

```java
// public
if (corePoolSize < 0 || maximumPoolSize < corePoolSize)  
    throw new IllegalArgumentException();  
int delta = corePoolSize - this.corePoolSize;  
this.corePoolSize = corePoolSize;
if (workerCountOf(ctl.get()) > corePoolSize)  
    interruptIdleWorkers();  
else if (delta > 0) {  
    // We don't really know how many new threads are "needed".  
    // As a heuristic, prestart enough new workers (up to new core size) to handle the current number 
    // of tasks in queue, but stop if queue becomes empty while doing so.
    int k = Math.min(delta, workQueue.size());  
    while (k-- > 0 && addWorker(null, true)) {  
        if (workQueue.isEmpty())  
            break;  
    }  
}  
```

```java
final ReentrantLock mainLock = this.mainLock;  
mainLock.lock();  
try {  
    for (Worker w : workers) {  
        Thread t = w.thread;  
        if (!t.isInterrupted() && w.tryLock()) {  
            try {  
                t.interrupt();  
            } catch (SecurityException ignore) {  
            } finally {  
                w.unlock();  
            }  
        }  
        if (onlyOne)  
            break;  
    }  
} finally {  
    mainLock.unlock();  
}  
```


# 数据查询

public getPoolSize()：int 返回线程池中的线程数量。
public getActiveCount()：int 返回还活跃着执行任务的线程数量。  Worker.isLock()
public getLargestPoolSize()：int返回曾经线程池中同时达到的最大线程数量。








# 构造函数

- 必须参数：核心线程数，最大线程数，线程存活时间，时间单位，BlockingQueue
- 可选参数：ThreadFactory，RejectedExecutionHandler

没有指定ThreadFactory时，使用默认值：Executors.defaultThreadFactory()
没有指定RejectedExecutionHandler时，使用默认值：new AbortPolicy()

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,  
                          BlockingQueue<Runnable> workQueue, 
                          ThreadFactory threadFactory,  
                          RejectedExecutionHandler handler){
	...
}
```





# 公有方法

## 核心方法

1）public execute(Runnable command) : void

在将来某个时候执行给定任务 `command` 。该任务可以在新线程或现有池线程中执行。如果由于此执行器已关闭或已达到其容量而无法提交任务以供执行，则该任务将由当前RejectedExecutionHandler处理。



-- --
关闭：

2）public shutdown()：void

执行一个有序的关闭，执行先前提交的任务，但不接受新任务。

如果已经关闭，则调用没有额外的效果。

3）public shutdownNow()：List< Runnable > 

尝试停止所有正在执行的任务，停止等待任务的处理，并返回等待执行的任务列表。从该方法返回时，这些任务将从任务队列中排出（移除）。

此方法不等待主动执行的任务终止。使用awaitTermination完成此操作。

除了尽力停止处理正在执行的任务之外，没有任何保证。此实现通过 Thread.interrupt()完成。任何无法响应中断的任务可能永远不会终止。

4）public boolean awaitTermination(long timeout, TimeUnit unit)





## public execute(Runnable command) : void



分三步：

1.
```java
public void execute(Runnable command) {  
    if (command == null)  
        throw new NullPointerException();  
    /*  
     * Proceed in 3 steps:
     *     
     * 1. If fewer than corePoolSize threads are running, try to start a new thread with the given 
     *    command as its first task.The call to addWorker atomically checks runState and workerCount,
	 *    and so prevents false alarms that would add threads when it shouldn't, by returning false.
     * 2. If a task can be successfully queued, then we still need to double-check whether we should 
     *    have added a thread(because existing ones died since last checking) or that the pool shut 
     *    down since entry into this method. So we recheck state and if necessary roll back the 
     *    enqueuing if stopped, or start a new thread if there are none. 
	 * 3. If we cannot queue task, then we try to add a new thread.If it fails, we know we are shut 
     *    down or saturated and so reject the task.
     */    
    int c = ctl.get();  
    if (workerCountOf(c) < corePoolSize) {  
        if (addWorker(command, true))  
            return;  
        c = ctl.get();  
    }  
    // 
    if (isRunning(c) && workQueue.offer(command)) {  
        int recheck = ctl.get();  
        if (! isRunning(recheck) && remove(command))  
            reject(command);  
        else if (workerCountOf(recheck) == 0)  
            addWorker(null, false);
    }  
    else if (!addWorker(command, false))
        reject(command);  
}
```







竞争导致条件是否发生改变的分析：

第一处：能保证。由程序逻辑保证。
第二处：不能保证。因为其他线程可能修改corePoolSize和maximumPoolSize。存在因其他线程对核心线程数和最大线程数进行扩容，但是局部变量仍然大于的问题。需要在函数外部处理。
第三处：不能保证。存在增加后线程数大于核心线程数或最大线程数的风险。需要在函数内部处理。

```java
private void addWorkerFailed(Worker w) {  
    final ReentrantLock mainLock = this.mainLock;  
    mainLock.lock();  
    try {  
        if (w != null)  
            workers.remove(w);  
        decrementWorkerCount();  
        tryTerminate();  
    } finally {  
        mainLock.unlock();  
    }  
}

final void tryTerminate() {  
    for (;;) {  
        int c = ctl.get();  
        if (isRunning(c) ||  
            runStateAtLeast(c, TIDYING) ||  
            (runStateLessThan(c, STOP) && ! workQueue.isEmpty()))  
            return;  
        if (workerCountOf(c) != 0) { // Eligible to terminate  
            interruptIdleWorkers(ONLY_ONE);  
            return;        }  
  
        final ReentrantLock mainLock = this.mainLock;  
        mainLock.lock();  
        try {  
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {  
                try {  
                    terminated();  
                } finally {  
                    ctl.set(ctlOf(TERMINATED, 0));  
                    termination.signalAll();  
                }  
                return;  
            }  
        } finally {  
            mainLock.unlock();  
        }  
        // else retry on failed CAS  
    }  
}
```





# 私有方法



![[Pasted image 20221215231633.png]]



```java
private void interruptIdleWorkers(boolean onlyOne) {  
    final ReentrantLock mainLock = this.mainLock;  
    mainLock.lock();  
    try {  
        for (Worker w : workers) {  
            Thread t = w.thread;  
            if (!t.isInterrupted() && w.tryLock()) {  
                try {  
                    t.interrupt();  
                } catch (SecurityException ignore) {  
                } finally {  
                    w.unlock();  
                }  
            }  
            if (onlyOne)  
                break;  
        }  
    } finally {  
        mainLock.unlock();  
    }  
}
```


## addWorker

	根据当前池的状态和给定的core或maximum约束来确定能否添加一个新的worker。
	如果可以添加，那么会相应改变worker的数量，创建、启动，把firstTask作为它的第一个任务开始运行。

- 参数：Runnable firstTask，boolean core
- 返回值：boolean

代码：

第一部分：

```java
// 只有状态处于RUNNING || (SHUTDOWN && firstTask为空 && 任务队列不空) 时才可能将线程数+1
retry:  
for (int c = ctl.get();;) {  
    // 状态 == STOP||TIDYING||TERMINATED 返回false
    // 状态 == SHUTDOWN && firstTask!=null && workQueue==null 返回false
    if (runStateAtLeast(c, SHUTDOWN)                                                     // 1
        && (runStateAtLeast(c, STOP) || firstTask != null || workQueue.isEmpty()))
        return false    // CAS循环
    for (;;) {  
        // 如果与corePoolSize(核心线程数)比较，比核心线程数大，返回false
        // 如果与maximumPoolSize(最大线程数)比较，比最大线程数大，返回false
        if (workerCountOf(c) >= ((core ? corePoolSize : maximumPoolSize) & COUNT_MASK))  // 2  
            return false;
        if (compareAndIncrementWorkerCount(c))
            break retry;                                                                 // 3
        c = ctl.get();  // Re-read ctl
        if (runStateAtLeast(c, SHUTDOWN))  // 如果有其他线程发起SHUTDOWN命令
            continue retry;
    }
}
```

第二部分：

```java
boolean workerStarted = false;
boolean workerAdded = false;
Worker w = null;  
try {  
    w = new Worker(firstTask);  
    final Thread t = w.thread;  
    if (t != null) {  
        final ReentrantLock mainLock = this.mainLock;  
        mainLock.lock();  
        try {  
            // Recheck while holding lock.  
            // Back out on ThreadFactory failure or if shut down before lock acquired.                
            int c = ctl.get();
            if (isRunning(c) || (runStateLessThan(c, STOP) && firstTask == null)) {  
                if (t.getState() != Thread.State.NEW)  
                    throw new IllegalThreadStateException();  
                workers.add(w);
                workerAdded = true;
                int s = workers.size();  
                if (s > largestPoolSize)  
                    largestPoolSize = s;  
            }  
        } finally {
            mainLock.unlock();
        }  
        if (workerAdded) {  
            t.start();  
            workerStarted = true;  
        }  
    }  
} finally {  
    if (! workerStarted)  
        addWorkerFailed(w);  
}  
return workerStarted;  
```



# 考虑问题

