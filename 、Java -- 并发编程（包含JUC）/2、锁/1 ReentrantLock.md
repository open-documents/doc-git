
可重入互斥锁（reentrant mutual exclusion Lock），与synchronized关键字具有相同语义，但是有更多的功能。

<span style="color:#00E0FF">ReentrantLock</span>重要方法<span style="color:#FF5500">lock()</span>、<span style="color:#FF5500">unlock()</span>通过<span style="color:#FFDD00">Sync</span>（<span style="color:#00E0FF">ReentrantLock</span>内部AQS的抽象子类）对象来完成具体功能。

```java
public class ReentrantLock implements Lock, java.io.Serializable {
	private final Sync sync;

	public void lock() {  sync.lock();  }
	public void unlock() {  sync.release(1);  }
}	
```

# lock()

下面对<span style="color:#00E0FF">ReentrantLock</span>#<span style="color:#FF5500">lock()</span>源码展开介绍，从前面可知<span style="color:#00E0FF">ReentrantLock</span>#<span style="color:#FF5500">lock()</span>实际调用的是<span style="color:#FFDD00">Sync</span>#<span style="color:#FF5500">lock()</span>，因此下面从<span style="color:#FFDD00">Sync</span>#<span style="color:#FF5500">lock()</span>开始介绍。

```java
/**
 * Sync
 */
final void lock() {  
    if (!initialTryLock())  
        acquire(1);  
}
```

<span style="color:#FFDD00">Sync</span>#<span style="color:#FF5500">lock()</span>分为2部分：initialTryLock()、acquire(1)。

---
initialTryLock()是<span style="color:#FFDD00">Sync</span>自身提供的抽象方法，由子类<span style="color:#FFC0CB">FairSync</span>（公平锁）、<span style="color:#FFC0CB">NonfairSync</span>（非公平锁）重写。下面以<span style="color:#FFC0CB">FairSync</span>#initialTryLock()展开介绍。

）检查锁状态，如果当前锁状态为0，说明当前锁还没有被其它任何线程占有。

如果下面2个条件都满足，则当前线程获取锁。
- <span style="color:#44cf57">查看之前是否有其它线程等待获取该锁</span>（hasQueuedThreads()，<span style="color:#44cf57">这是公平锁与非公平锁的主要区别</span>，关于<span style="color:#FFC0CB">NonfairSync</span>#initialTryLock()见<a href="#1">附录</a>）
- 以CAS方式设置锁状态为1，因为尽管已经检查过锁状态为1，但是在执行CAS设置时，其它线程已经修改了锁状态，从而导致当前锁同样需要等待。

）检查锁状态，如果当前锁状态不为0，说明当前锁已经被其它线程占有。

需要检查占有该锁的线程是否是当前线程（getExclusiveOwnerThread()），如果是当前线程，则将锁状态加1（释放琐时-1）实现重入效果。

```java
/** 
 * FairSync
 */
final boolean initialTryLock() {  
    Thread current = Thread.currentThread();  
    int c = getState();  
    if (c == 0) {  
	    // 1. 在当前线程前有其它线程等待获取锁
	    // 2. 其它线程没有占有锁
        if (!hasQueuedThreads() && compareAndSetState(0, 1)) {  
            setExclusiveOwnerThread(current);  
            return true;  
        }  
    } 
    else if (getExclusiveOwnerThread() == current) {  // 重入
        if (++c < 0) // overflow  
            throw new Error("Maximum lock count exceeded");  
        setState(c);  
        return true;  
    }  
    return false;  
}
```


---

```java
protected final boolean tryAcquire(int acquires) {  
    if (getState() == 0 && !hasQueuedPredecessors() &&  
        compareAndSetState(0, acquires)) {  
        setExclusiveOwnerThread(Thread.currentThread());  
        return true;  
    }  
    return false;  
}
```

```java
protected final boolean tryAcquire(int acquires) {  
    if (getState() == 0 && compareAndSetState(0, acquires)) {  
        setExclusiveOwnerThread(Thread.currentThread());  
        return true;  
    }  
    return false;  
}
```




-- --
# 公平锁、非公平锁

根据Sync的具体实现类，ReentrantLock的构造函数提供2类锁：公平锁和非公平锁。

```java
public ReentrantLock() {  
    sync = new NonfairSync();                             
}
public ReentrantLock(boolean fair) {  
    sync = fair ? new FairSync() : new NonfairSync();  
}
```

非公平锁：在锁分配给活动线程时，不能保证分配等待时间最长的线程，因此可能出现较大的饥饿问题。
公平锁：在锁分配给活动线程时，一定分配给等待时间最长的线程。根据源码描述，<font color=44cf57>公平锁会使整体吞吐量下降很多</font>，需要注意的是，公平锁并不能完全避免线程饥饿的问题（关于具体原因，见后文）。

如何理解：<font color="00E0FF">锁的公平性并不能保证线程调度的公平性，因此一个线程可能连续多次获得锁？</font>

公平锁和非公平锁都是针对于活动线程之间竞争锁的分配策略，而线程的活动与否是由操作系统决定（即线程的调度），因此可能出现下面的情况。

线程A一直被操作系统调度，而线程B一直等待，从而导致线程B一直不能处于活动状态，从而线程B没有参与锁竞争的资格，所以即使线程B等待时间最长，但是获得不了锁，但是一旦操作系统调度了线程B，线程B（相对于线程A)一定能获得公平锁。

-- -- 
# 最佳实践（来源于源码）

建议在调用后立即使用try块锁定，最常见的是在before/after构造中，例如：

```java
class X {
   private final ReentrantLock lock = new ReentrantLock();
   // ...

   public void m() {
     lock.lock();  // block until condition holds
     try {
       // ... method body
     } finally {
       lock.unlock();
     }
   }
 }
```

-- --
# lock()、lockInterruptibly()

1）lock()：void

当前线程（记为A）尝试获取锁，当没有其它线程持有当前锁时，获取锁，直接返回。（与lockInterruptibly()相同）
当前线程（记为A）尝试获取锁，当锁的持有者就是线程A时，持有数量加1，直接返回。（与lockInterruptibly()相同）
当前线程（记为A）尝试获取锁，当锁被其它线程持有时，线程A一直休眠到获取锁时。

2）lockInterruptibly()：void

当前线程（记为A）尝试获取锁，当没有其它线程持有当前锁时，获取锁，直接返回。（与lock()相同）
当前线程（记为A）尝试获取锁，当锁的持有者就是线程A时，持有数量加1，直接返回。（与lock()相同）
当前线程（记为A）尝试获取锁，当锁被其它线程持有时，线程A一直休眠到获取锁或被其它线程打断时。

当线程A在进入lockInterruptibly()方法前，设置了interrupted标识，抛出InterruptedException，并且清空interrupted标识。
当线程A在执行lockInterruptibly()方法过程中，被其它线程打断，抛出InterruptedException，并且清空interrupted标识。

# tryLock()


# unlock()








）unlock() : void 

尝试着释放锁。

如果当前线程是锁的持有者，那么执行一次这个方法， `hold count` 减 1，减到了 0 ，就彻底释放锁。

如果当前线程不是锁的持有者，抛出 `IllegalMonitorStateException `异常。




# 附录

## <a id="1">NonFairSync#initialTryLock()</a>

下面介绍NonFairSync#initialTryLock()
```java
/**
 * NonFairSync
 */
final boolean initialTryLock() {  
    Thread current = Thread.currentThread();  
    if (compareAndSetState(0, 1)) { // first attempt is unguarded  
        setExclusiveOwnerThread(current);  
        return true;  
    } 
    else if (getExclusiveOwnerThread() == current) {  
        int c = getState() + 1;  
        if (c < 0) // overflow  
            throw new Error("Maximum lock count exceeded");  
        setState(c);  
        return true;  
    } 
    else  
        return false;  
}
```