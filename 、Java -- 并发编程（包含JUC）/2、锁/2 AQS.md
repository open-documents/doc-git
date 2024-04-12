
<span style="color:#44cf57">AQS（AbstractQueuedSynchronizer）</span>是一个抽象类。AQS提供了锁和同步工具的一部分通用功能的实现，使用AQS可以更加简单地创建锁（如ReentrantLock）和同步工具（


# 获取锁

从<span style="color:#00E0FF">ReentrantLock</span>#<span style="color:#FF5500">lock()</span>可知真正获取锁的过程是<span style="color:#44cf57">AQS</span>#<span style="color:#FF5500">acquire()</span>，因此下面对锁的获取过程做详细介绍。
```java
/**
 * Class: AQS
 */
public final void acquire(int arg) {  
	// tryAcquire(): protected方法, 默认抛出UnsupportedOperationException()异常
    if (!tryAcquire(arg))  
        acquire(null, arg, false, false, false, 0L);  
}
```
1）tryAcquire()：尝试获取锁，默认为No Op Protected方法，由子类重写。

2）acquire()

首先介绍<span style="color:#44cf57">AQS</span>#<span style="color:#FF5500">acquire()</span>中出现的形参、局部变量。
- node -- 当前线程对应的节点
- pred -- 

first -- boolean类型，当前节点是否是队列中的第一个节点，默认为false。只有在当前线程对应节点入队后，通过当前线程对应节点的前驱节点是否是头结点来修改first值。
```java
final int acquire(Node node, int arg, boolean shared, boolean interruptible, boolean timed, long time) {  
	// ... 其它代码
	for(;;) {
		if (!first && (pred = (node == null) ? null : node.prev) != null &&  
		    !(first = (head == pred)))  // first 通过当前线程对应节点的前驱节点是否是头结点判断
		    {
		    
		}
		// ... 其它代码
	}
```


---
第一次循环：

1）尝试获取锁。
```java
final int acquire(Node node, int arg, boolean shared, boolean interruptible, boolean timed, long time) {  
	// ... 其它代码
	for(;;) {
		// ... 其它代码
        if (first || pred == null) {  
		    boolean acquired;  
		    try {  
		        if (shared)  
		            acquired = (tryAcquireShared(arg) >= 0);  
		        else  
		            acquired = tryAcquire(arg);  
		    } 
		    catch (Throwable ex) {  
		        cancelAcquire(node, interrupted, false);  
		        throw ex;  
		    }  
		    if (acquired) {  
		        if (first) {  
		            node.prev = null;  
		            head = node;  
		            pred.next = null;  
		            node.waiter = null;  
		            if (shared)  
		                signalNextIfShared(node);  
		            if (interrupted)  
		                current.interrupt();  
		        }  
		        return 1;  
		    }  
		}
		// ... 其它代码
	// ... 其它代码
}
```
2）如果没有获取到锁，为当前线程创建节点。
```java
final int acquire(Node node, int arg, boolean shared, boolean interruptible, boolean timed, long time) {  

	for(;;) 
		// ... 其它代码
        if (node == null) {                 // allocate; retry before enqueue  
            if (shared)  
                node = new SharedNode();  
            else                
	            node = new ExclusiveNode();  
        } 
		// ... 其它代码
}
```

---
第二次循环

1）尝试获取锁。（同第一次循环）

2）当前线程对应节点入队。
```java
final int acquire(Node node, int arg, boolean shared, boolean interruptible, boolean timed, long time) { 
	// ... 其它代码
	for(;;){
		// ... 其它代码
        if (node == null) {...} 
        // 节点还没有入队, 则尝试入队
        else if (pred == null) {          // try to enqueue  
            node.waiter = current;  
            Node t = tail;  
            node.setPrevRelaxed(t);         // CAS方式设置当前前驱节点, avoid unnecessary fence  
            if (t == null)                  // 
                tryInitializeHead();        // 初始化队列
            else if (!casTail(t, node))     // CAS方式将当前节点设置为尾节点TAIL=node
                node.setPrevRelaxed(null);  // back out  
            else  
                t.next = node;  
        } 
        // ... 其它代码
    }
    // ... 其它代码
}
```

---
第三次循环

对于队列中的第一个节点来说。

1）尝试获取锁。

2）如果没有获取到锁，修改线程对应节点状态。

```java
final int acquire(Node node, int arg, boolean shared, boolean interruptible, boolean timed, long time) { 
	// ... 其它代码
	for(;;){
		// ... 其它代码
		if (node == null) {}
		else if (pred == null) {}
		else if (first && spins != 0) {}
		else if (node.status == 0) {  
		    node.status = WAITING;          // enable signal and recheck  
		} 
		else {}
    }
    // ... 其它代码
}
```

对于队列中的非第一个节点来说。

1）
```java
final int acquire(Node node, int arg, boolean shared, boolean interruptible, boolean timed, long time) { 
	// ... 其它代码
	for(;;){
		if (!first && (pred = (node == null) ? null : node.prev) != null &&  
		    !(first = (head == pred))) {  
		    if (pred.status < 0) {  
		        cleanQueue();           // predecessor cancelled  
		        continue;  
		    } 
		    else if (pred.prev == null) {  
		        Thread.onSpinWait();    // ensure serialization  
		        continue;  
		    }  
		}
        // ... 其它代码
    }
    // ... 其它代码
}
```
2）修改线程对应节点状态。


---
第四次循环

对于第一个节点来说。

1）尝试获取锁。
2）
```java
final int acquire(Node node, int arg, boolean shared, boolean interruptible, boolean timed, long time) { 
	// ... 其它代码
	for(;;){
		// ... 其它代码
		if (node == null) {}
		else if (pred == null) {}
		else if (first && spins != 0) {}
		else if (node.status == 0) {} 
		else {
			long nanos;  
			spins = postSpins = (byte)((postSpins << 1) | 1);  
			if (!timed)  
			    LockSupport.park(this);  
			else if ((nanos = time - System.nanoTime()) > 0L)  
			    LockSupport.parkNanos(this, nanos);  
			else  
			    break;  
			node.clearStatus();  
			if ((interrupted |= Thread.interrupted()) && interruptible)  
			    break;
		}
    }
    // ... 其它代码
}
```

对于非第一个节点来说。


---
第五次循环

对于第一个节点来说。

1）尝试获取锁。

2）
```java
final int acquire(Node node, int arg, boolean shared, boolean interruptible, boolean timed, long time) { 
	// ... 其它代码
	for(;;){
		// ... 其它代码
		if (node == null) {}
		else if (pred == null) {}
		else if (first && spins != 0) {
			--spins;                        // reduce unfairness on rewaits  
			Thread.onSpinWait();
		}
		else if (node.status == 0) {} 
		else {}
    }
    // ... 其它代码
}
```











# 释放锁


```java
public void unlock() {  
    sync.release(1);  
}
```

```java
public final boolean release(int arg) {  
    if (tryRelease(arg)) {  
        signalNext(head);  
        return true;  
    }  
    return false;  
}
```

```java
/**
 * Class: Sync
 */
protected final boolean tryRelease(int releases) {  
    int c = getState() - releases;  
    if (getExclusiveOwnerThread() != Thread.currentThread())  
        throw new IllegalMonitorStateException();  
    boolean free = (c == 0);  
    if (free)  
        setExclusiveOwnerThread(null);  
    setState(c);  
    return free;  
}
```

```java
/**
 * Class: AQS
 * Description: 
 *   唤醒指定节点h的后继节点, 
 *   Wakes up the successor of given node, if one exists, and unsets its WAITING status to avoid park race. This may fail to wake up an eligible thread when one or more have been cancelled, but cancelAcquire ensures liveness.
 */
private static void signalNext(Node h) {  
    Node s;  
    if (h != null && (s = h.next) != null && s.status != 0) {  
        s.getAndUnsetStatus(WAITING);  
        LockSupport.unpark(s.waiter);  
    }  
}
```

# AQS中的队列

AQS使用HEAD、TAIL两个属性来维护一个队列。

1）队列为空

首先看队列的初始状态（即队列为空）的情况。AQS使用 tryInitializeHead() 初始化整个队列。
```java
private void tryInitializeHead() {  
    Node h = new ExclusiveNode();  
    if (U.compareAndSetReference(this, HEAD, null, h))  
        tail = h;  
}
```
不难看出<font color=44cf57>队列为空时，HEAD=TAIL，都指向同一个不持有数据的节点。</font>
```text
    HEAD         TAIL
      ↓           ↓
	+---------------+   
	| ExclusiveNode |  
	+---------------+  
```

2）节点入队

第一行代码及状态转换。
```java
Node t = tail;  

/*

head(HEAD)      tail(TAIL)        head    tail      t
    ↓               ↓               ↓       ↓       ↓
    +---------------+               +---------------+
    | ExclusiveNode |        ->     | ExclusiveNode |
    +---------------+               +---------------+
    
*/
```
第二行代码及状态转换。
```java
node.setPrevRelaxed(t);

/*
head    tail      t       
  ↓       ↓       ↓     prev  
  +---------------+   <-------   +------+
  | ExclusiveNode |              | Node |
  +---------------+              +------+
  
*/
```   

```java
// 队列还没有初始化
if (t == null)
	tryInitializeHead();  
else if (!casTail(t, node))     // U.compareAndSetReference(this, TAIL, tail, node), 防止tail已经被其它线程修改
	node.setPrevRelaxed(null);  // back out  
else  
	t.next = node;
```

```java

/*
head              t                tail                        
  ↓               ↓     prev         ↓  
  +---------------+   <-------   +------+
  | ExclusiveNode |              | Node |
  +---------------+   ------->   +------+
                        next
*/

```




3）节点出队

下面介绍源码中节点入队的代码部分。
```java
node.prev = null;  
head = node;  
pred.next = null;  
node.waiter = null; 
```


# 八股问题

<span style="color:#fb463c">问题1：什么是独占锁？AQS是否是独占锁？</span>

<span style="color:#44cf57">独占锁</span>是指当锁被某一线程获取后，其它线程再申请获取该锁时，无法获取。

AQS是独占锁，AQS继承了AbstractOwnableSynchronizer类，其中重要且唯一的属性是exclusiveOwnerThread（Thread类型），表示当前占有锁的线程。
```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer

public abstract class AbstractOwnableSynchronizer{
	private transient Thread exclusiveOwnerThread;
	protected final void setExclusiveOwnerThread(Thread thread) {exclusiveOwnerThread = thread;}  
}
```
当线程调用AQS的子类（如ReentrantLock）中的lock()方法获取到锁后，会修改exclusiveOwnerThread属性为当前线程。

---

-- --
# 附录

## 

```java
/**  
 * Class: AQS
 */
 final int acquire(Node node, int arg, boolean shared, boolean interruptible, boolean timed, long time) {  

    Thread current = Thread.currentThread();  
    byte spins = 0, postSpins = 0;   // retries upon unpark of first thread  
    boolean interrupted = false, first = false;  
    Node pred = null;                // predecessor of node when enqueued  
  
    /*     
     * Repeatedly:     
     *  Check if node now first     
     *    if so, ensure head stable, else ensure valid predecessor     
     *  if node is first or not yet enqueued, try acquiring     
     *  else if node not yet created, create it
     *  else if not yet enqueued, try once to enqueue     
     *  else if woken from park, retry (up to postSpins times)     
     *  else if WAITING status not set, set and retry     
     *  else park and clear WAITING status, and check cancellation   
	 */  
    for (;;) {  
        if ( !first                                                          
	         && (pred = (node == null) ? null : node.prev) != null             
             && !(first = (head == pred))
            ) {  
            if (pred.status < 0) {  
                cleanQueue();           // predecessor cancelled  
                continue;  
            } else if (pred.prev == null) {  
                Thread.onSpinWait();    // ensure serialization  
                continue;  
            }  
        }  

        if (first || pred == null) {  
            boolean acquired;  
            try {  
                if (shared)  
                    acquired = (tryAcquireShared(arg) >= 0);  
                else                    
	                acquired = tryAcquire(arg);  
            } catch (Throwable ex) {  
                cancelAcquire(node, interrupted, false);  
                throw ex;  
            }

            if (acquired) {  
                if (first) {  
                    node.prev = null;  
                    head = node;  
                    pred.next = null;  
                    node.waiter = null;  
                    if (shared)  
                        signalNextIfShared(node);  
                    if (interrupted)  
                        current.interrupt();  
                }  
                return 1;  
            }  
        }  

        if (node == null) {                 // allocate; retry before enqueue  
            if (shared)  
                node = new SharedNode();  
            else                
	            node = new ExclusiveNode();  
        } 
        else if (pred == null) {          // try to enqueue  
            node.waiter = current;  
            Node t = tail;  
            node.setPrevRelaxed(t);         
            if (t == null)                  
                tryInitializeHead();        
            else if (!casTail(t, node))     
                node.setPrevRelaxed(null);  
            else  
                t.next = node;  
        } 
        else if (first && spins != 0) {  
            --spins;                        // reduce unfairness on rewaits  
            Thread.onSpinWait();  
        } 
        else if (node.status == 0) {  
            node.status = WAITING;          // enable signal and recheck  
        } 
        else {  
            long nanos;  
            spins = postSpins = (byte)((postSpins << 1) | 1);  
            if (!timed)  
                LockSupport.park(this);  
            else if ((nanos = time - System.nanoTime()) > 0L)  
                LockSupport.parkNanos(this, nanos);  
            else                
	            break;            
	        node.clearStatus();  
            if ((interrupted |= Thread.interrupted()) && interruptible)  
                break;  
        }  
    }  
    return cancelAcquire(node, interrupted, interruptible);  
}
```

## 

```java
public final boolean hasQueuedPredecessors() {  
    Thread first = null; Node h, s;  
    if ((h = head) != null && ((s = h.next) == null ||  
                               (first = s.waiter) == null ||  
                               s.prev == null))  
        first = getFirstQueuedThread(); // retry via getFirstQueuedThread  
    return first != null && first != Thread.currentThread();  
}
```