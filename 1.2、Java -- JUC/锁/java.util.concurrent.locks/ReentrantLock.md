可重入互斥锁，其基本行为和语义与使用同步方法和语句访问的隐式监视器锁相同，但具有扩展功能。

ReentrantLock由上次成功锁定但尚未解锁的线程拥有。当锁不属于另一个线程时，调用锁的线程将返回并成功获取锁。如果当前线程已经拥有锁，则该方法将立即返回。这可以使用方法isHeldByCurrentThread()和getHoldCount()进行检查。

此类的构造函数接受可选的公平性参数。当设置为true时，在争用情况下，锁有利于授予对等待时间最长的线程的访问权。否则，此锁不能保证任何特定的访问顺序。与使用默认设置的程序相比，使用由许多线程访问的公平锁的程序可能显示较低的总吞吐量（即，较慢；通常较慢），但获得锁的时间差异较小，并保证不会出现饥饿。然而，请注意，锁的公平性不能保证线程调度的公平性。因此，使用公平锁的多个线程中的一个可以连续多次获得该锁，而其他活动线程没有进展并且当前没有持有该锁。还要注意，untimed tryLock（）方法不符合公平设置。如果锁可用，即使其他线程正在等待，它也会成功。

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

除了实现锁接口之外，该类还定义了许多用于检查锁状态的公共和受保护方法。其中一些方法仅适用于仪器和监控。

# 构造函数

默认创建非公平锁

```java
public ReentrantLock() {  
    sync = new NonfairSync();  
}
public ReentrantLock(boolean fair) {  
    sync = fair ? new FairSync() : new NonfairSync();  
}
```

# 公共方法

## 核心方法

）lock() : void

获取锁。

如果另一个线程未持有锁，则获取该锁并立即返回，将锁持有计数设置为1。

如果当前线程已经持有锁，则持有计数将增加1，方法立即返回。

如果锁由另一个线程持有，则当前线程出于线程调度目的而被禁用，并处于休眠状态，直到获取了锁，此时锁持有计数被设置为1。


）unlock() : void 

尝试着释放锁。

如果当前线程是锁的持有者，那么执行一次这个方法， `hold count` 减 1，减到了 0 ，就彻底释放锁。

如果当前线程不是锁的持有者，抛出 `IllegalMonitorStateException `异常。

## 属性方法

