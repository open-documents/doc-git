# 使用

提供了一个框架，用于实现依赖先进先出（FIFO）等待队列的阻塞锁和相关同步器（信号量、事件等）。此类被设计为大多数类型的同步器的有用基础，这些同步器依赖于单个原子int值来表示状态。子类必须定义更改此状态的受保护方法，并定义该状态在获取或释放此对象方面的含义。考虑到这些，这个类中的其他方法执行所有排队和阻塞机制。子类可以维护其他状态字段，但仅跟踪使用getState、setState和compareAndSetState方法操作的原子更新的int值。





要将此类用作同步器的基础，请通过使用getState、setState和/或compareAndSetState检查 和/或 修改同步状态，重新定义以下方法（如果适用）：

tryAcquire
tryRelease
tryAcquireShared
tryReleaseShared
isHeldExclusively

这些方法默认抛出UnsupportedOperationException异常。这些方法的实现必须是内部线程安全的，并且通常应该是短的而不是块的。定义这些方法是使用此类的唯一受支持的方法。所有其他方法都被声明为final，因为它们不能独立更改。
您还可以发现从AbstractOwnableSynchronizer继承的方法对于跟踪拥有独占同步器的线程非常有用。我们鼓励您使用它们——这使监控和诊断工具能够帮助用户确定哪些线程持有锁。
即使该类基于内部FIFO队列，它也不会自动强制执行FIFO获取策略。独占同步的核心形式如下：
```java
Acquire:
    while (!tryAcquire(arg)) {
       enqueue thread if it is not already queued;
       possibly block current thread;
    }
  
Release:
    if (tryRelease(arg))
       unblock the first queued thread;
```

（共享模式类似，但可能涉及级联信号。）

由于获取中的检查是在排队之前调用的，因此新获取的线程可能会先于其他被阻塞和排队的线程。但是，如果需要，您可以通过内部调用一个或多个检查方法来定义tryAcquire和/或tryAcquireShared以禁用讨价还价，从而提供公平的FIFO采集顺序。特别是，如果hasQueuedPredessors（一种专门设计用于公平同步器的方法）返回true，大多数公平同步器可以将tryAcquire定义为返回false。其他变化也是可能的。

默认讨价还价（也称为贪婪、放弃和车队回避）策略的吞吐量和可扩展性通常最高。虽然不能保证这是公平的或无饥饿的，但允许较早排队的线程在稍后排队的线程之前重新配置，并且每个重新配置都有公平的机会对传入线程成功。此外，虽然获取在通常意义上不“旋转”，但它们可能会执行多次tryAcquire调用，并在阻塞之前穿插其他计算。当独占同步仅短暂保持时，这提供了旋转的大部分好处，而当不保持时，则不会产生大部分负担。如果需要，您可以通过前面的调用来增强这一点，以获取具有“快速路径”检查的方法，可能会预检查hasContended和/或hasQueuedThreads，以便仅在同步器可能不争用时才这样做。


这里有一个不可重入互斥锁类，它使用值0表示解锁状态，使用值1表示锁定状态。虽然不可重入锁并不严格要求记录当前所有者线程，但无论如何，该类都会这样做，以使使用情况更易于监控。它还支持条件并公开一些检测方法：

```java
class Mutex implements Lock, java.io.Serializable {
   // Our internal helper class
   private static class Sync extends AbstractQueuedSynchronizer {
     // Acquires the lock if state is zero
     public boolean tryAcquire(int acquires) {
       assert acquires == 1; // Otherwise unused
       if (compareAndSetState(0, 1)) {
         setExclusiveOwnerThread(Thread.currentThread());
         return true;
       }
       return false;
     }

     // Releases the lock by setting state to zero
     protected boolean tryRelease(int releases) {
       assert releases == 1; // Otherwise unused
       if (!isHeldExclusively())
         throw new IllegalMonitorStateException();
       setExclusiveOwnerThread(null);
       setState(0);
       return true;
     }

     // Reports whether in locked state
     public boolean isLocked() {
       return getState() != 0;
     }

     public boolean isHeldExclusively() {
       // a data race, but safe due to out-of-thin-air guarantees
       return getExclusiveOwnerThread() == Thread.currentThread();
     }

     // Provides a Condition
     public Condition newCondition() {
       return new ConditionObject();
     }

     // Deserializes properly
     private void readObject(ObjectInputStream s)
         throws IOException, ClassNotFoundException {
       s.defaultReadObject();
       setState(0); // reset to unlocked state
     }
   }

   // The sync object does all the hard work. We just forward to it.
   private final Sync sync = new Sync();

   public void lock()              { sync.acquire(1); }
   public boolean tryLock()        { return sync.tryAcquire(1); }
   public void unlock()            { sync.release(1); }
   public Condition newCondition() { return sync.newCondition(); }
   public boolean isLocked()       { return sync.isLocked(); }
   public boolean isHeldByCurrentThread() {
     return sync.isHeldExclusively();
   }
   public boolean hasQueuedThreads() {
     return sync.hasQueuedThreads();
   }
   public void lockInterruptibly() throws InterruptedException {
     sync.acquireInterruptibly(1);
   }
   public boolean tryLock(long timeout, TimeUnit unit)
       throws InterruptedException {
     return sync.tryAcquireNanos(1, unit.toNanos(timeout));
   }
 }
```


这里有一个类似于CountDownLatch的锁存类，只是它只需要一个信号来触发。因为锁存器是非独占的，所以它使用共享的获取和释放方法。
```java
 class BooleanLatch {

   private static class Sync extends AbstractQueuedSynchronizer {
     boolean isSignalled() { return getState() != 0; }

     protected int tryAcquireShared(int ignore) {
       return isSignalled() ? 1 : -1;
     }

     protected boolean tryReleaseShared(int ignore) {
       setState(1);
       return true;
     }
   }

   private final Sync sync = new Sync();
   public boolean isSignalled() { return sync.isSignalled(); }
   public void signal()         { sync.releaseShared(1); }
   public void await() throws InterruptedException {
     sync.acquireSharedInterruptibly(1);
   }
 }
```



# 属性

**1）state属性**

```java
private static final long STATE = U.objectFieldOffset(AbstractQueuedSynchronizer.class, "state");

private volatile int state;  // The synchronization state.

protected final int getState() {return state;}
protected final void setState(int newState) {state = newState;}
protected final boolean compareAndSetState(int expect, int update) {
	return U.compareAndSetInt(this, STATE, expect, update);  
}
```

**2）head && tail**

```java
private static final long HEAD = U.objectFieldOffset(AbstractQueuedSynchronizer.class, "head");  
private static final long TAIL = U.objectFieldOffset(AbstractQueuedSynchronizer.class, "tail");

private transient volatile Node head;  //等待队列的头
private transient volatile Node tail;  //等待队列的尾
```


# 公有方法

## 核心方法



## 查询方法

1）hasQueuedThreads() : boolean

功能：查询是否有线程等待 `acquire()`

因为是从尾到头对节点的状态进行检查（读操作），而对线程队列并没有读写的互斥，因此返回true只能说明可能存在线程等待着 `acquire()`（因为取消 `acquire()` 随时可能发生）。

**2）hasContended() : bool**

功能：查询是否存在线程正在竞争获得synchronizer。

因为是直接对头节点的状态进行判断，所以能在常量时间内返回，同时返回true，说明一定存在线程正在竞争。



# 需要重写方法(protected方法)

## 1.1）tryAcquire(int arg) : boolean

- 尝试以 `exclusive` 模式获取锁。默认实现不支持这种操作，默认抛出  `UnsupportedOperationException` 异常。
- 该方法在acquire()方法的内部被调用。
- 该方法可以用于实现Lock.tryLock()。

![[Pasted image 20221219022313.png]]

## 1.2）tryRelease(int arg) : boolean

- 尝试修改 `state` ，以反应释放锁。默认实现不支持这种操作，抛出  `UnsupportedOperationException` 异常。
- 该方法在release()方法的内部被调用。
- 该方法可以用于实现Lock.tryUnLock()。


2.1）




# CLH Node

```java
abstract static class Node{

volatile Node prev;       // initially attached via casTail  
volatile Node next;       // visibly nonnull when signallable  
Thread waiter;            // visibly nonnull when enqueued  
volatile int status;      // written by owner, atomic bit ops by others


final boolean casPrev(Node c, Node v) {  // for cleanQueue  
    return U.weakCompareAndSetReference(this, PREV, c, v);  
}  
final boolean casNext(Node c, Node v) {  // for cleanQueue  
    return U.weakCompareAndSetReference(this, NEXT, c, v);  
}  
final int getAndUnsetStatus(int v) {     // for signalling  
    return U.getAndBitwiseAndInt(this, STATUS, ~v);  
}  
final void setPrevRelaxed(Node p) {      // for off-queue assignment  
    U.putReference(this, PREV, p);  
}  
final void setStatusRelaxed(int s) {     // for off-queue assignment  
    U.putInt(this, STATUS, s);  
}  
final void clearStatus() {               // for reducing unneeded signals  
    U.putIntOpaque(this, STATUS, 0);  
}

private static final long STATUS = U.objectFieldOffset(Node.class, "status");  
private static final long NEXT   = U.objectFieldOffset(Node.class, "next");  
private static final long PREV   = U.objectFieldOffset(Node.class, "prev");

}
```

## CLH Node实现类

static final class ExclusiveNode extends Node { }  
static final class SharedNode extends Node { }


# FairSync

**1）initialTryLock()**

```java
final boolean initialTryLock() {  
    Thread current = Thread.currentThread();  
    int c = getState();  
    if (c == 0) {
        if (!hasQueuedThreads() && compareAndSetState(0, 1)) {  
            setExclusiveOwnerThread(current);  
            return true;
        }  
    }
    else if (getExclusiveOwnerThread() == current) {  
        if (++c < 0) // overflow  
            throw new Error("Maximum lock count exceeded");  
        setState(c);  
        return true;
    }  
    return false;  
}
```

2）protected final tryAcquire(int acquires) : boolean

```java
protected final boolean tryAcquire(int acquires) {  
    if (getState() == 0 && !hasQueuedPredecessors() && compareAndSetState(0, acquires)) {  
        setExclusiveOwnerThread(Thread.currentThread());  
        return true;
    }
    return false;  
}
```

# NonfairSync

```java
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


Acquire for non-reentrant cases after initialTryLock prescreen
```java
protected final boolean tryAcquire(int acquires) {  
    if (getState() == 0 && compareAndSetState(0, acquires)) {  
        setExclusiveOwnerThread(Thread.currentThread());  
        return true;
    }  
    return false;  
}
```