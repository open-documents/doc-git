

两个实现类：

Condition将Object监视器方法（wait、notify和notifyAll）分解为不同的对象，通过将它们与任意Lock实现的使用相结合，为每个对象提供多个等待集的效果。当Lock代替了同步方法和语句的使用时，Condition代替了Object监视器方法的使用。

Conditions（也称为条件队列或条件变量）为一个线程提供了暂停执行（“等待”）的方法，直到另一个线程通知某个状态条件现在可能为真。
由于对共享状态信息的访问发生在不同的线程中，因此必须对其进行保护，因此某种形式的锁与条件相关联。等待条件提供的关键属性是，它自动释放关联的锁并挂起当前线程，就像Object.wait一样。

例如，假设我们有一个支持put和take方法的有界缓冲区。如果在一个空缓冲区上尝试take，那么线程将阻塞，直到一个项目变为可用；如果试图在满缓冲区上放置，那么线程将阻塞，直到有空间可用。我们希望在单独的等待集中继续等待put线程和take线程，以便在缓冲区中的项目或空间变为可用时，我们可以使用只通知单个线程的优化。这可以使用两个条件实例来实现。
```java
class BoundedBuffer<E> {
	final Lock lock = new ReentrantLock();
	final Condition notFull  = lock.newCondition(); 
	final Condition notEmpty = lock.newCondition(); 
	
	final Object[] items = new Object[100];
	int putptr, takeptr, count;
	
	public void put(E x) throws InterruptedException {
	  lock.lock();
	  try {
	    while (count == items.length)
	      notFull.await();
	    items[putptr] = x;
	    if (++putptr == items.length) putptr = 0;
	    ++count;
	    notEmpty.signal();
	  } finally {
	    lock.unlock();
	  }
	}
  
	public E take() throws InterruptedException {
	  lock.lock();
	  try {
	    while (count == 0)
	      notEmpty.await();
	    E x = (E) items[takeptr];
	    if (++takeptr == items.length) takeptr = 0;
	    --count;
	    notFull.signal();
	    return x;
	  } finally {
	    lock.unlock();
	  }
	}
   }
```
（java.util.concurrent.ArrayBlockingQueue类提供了此功能，因此没有理由实现此示例用法类。）

条件实现可以提供不同于对象监视方法的行为和语义，例如保证通知的顺序，或者在执行通知时不需要锁。如果一个实现提供了这种专门的语义，那么该实现必须记录这些语义。

注意，Condition实例只是普通对象，它们本身可以用作同步语句中的目标，并且可以调用它们自己的监视器等待和通知方法。获取条件实例的监视锁或使用其监视方法与获取与该条件相关联的锁或使用它的等待和信令方法没有指定的关系。建议您不要以这种方式使用Condition实例，除非是在它们自己的实现中。

除非另有说明，否则为任何参数传递空值将导致引发NullPointerException。




在等待条件时，通常允许发生“虚假唤醒”，作为对底层平台语义的让步。这对大多数应用程序几乎没有实际影响，因为条件应该始终在循环中等待，测试等待的状态谓词。实现可以自由地消除虚假唤醒的可能性，但建议应用程序程序员始终假设它们可能发生，因此始终在循环中等待。

条件等待的三种形式（可中断、不可中断和定时）在某些平台上的易实现性及其性能特征方面可能有所不同。特别是，可能很难提供这些特性并维护特定的语义，例如排序保证。此外，在所有平台上实现中断线程的实际暂停的能力可能并不总是可行的。

因此，实现不需要为所有三种形式的等待定义完全相同的保证或语义，也不需要支持线程实际暂停的中断。

一个实现需要清楚地记录每个等待方法提供的语义和保证，当一个实现确实支持线程暂停的中断时，它必须遵守该接口中定义的中断语义。

As interruption generally implies cancellation, and checks for interruption are often infrequent, an implementation can favor responding to an interrupt over normal method return. This is true even if it can be shown that the interrupt occurred after another action that may have unblocked the thread. An implementation should document this behavior.

由于中断通常意味着取消，并且对中断的检查通常不频繁，因此实现可能倾向于响应中断而不是正常的方法返回。即使可以显示中断发生在另一个可能已取消阻止线程的操作之后，也是如此。实现应记录此行为。

## 方法

void await() throws InterruptedException;

与此条件相关联的锁被自动释放，当前线程出于线程调度目的被禁用，并处于休眠状态，直到发生以下四种情况之一：
- 其他一些线程调用此条件的信号方法，而当前线程恰好被选为要唤醒的线程；
- 其他一些线程为此Condition调用signalAll方法
- 其他一些线程中断当前线程，支持中断线程暂停
- 出现“虚假唤醒”
在所有情况下，在该方法返回之前，当前线程必须重新获取与该条件相关联的锁。当线程返回时，它保证持有此锁。
如果当前线程：
- 在进入该方法时设置其中断状态；或
- 在等待时被中断，并且支持线程暂停的中断，

当调用此方法时，假定当前线程持有与此Condition关联的锁。这取决于实施情况，以确定是否是这种情况，如果不是，如何应对。通常，将抛出异常（例如IllegalMonitorStateException），并且实现必须记录这一事实。
与响应信号的正常方法返回相比，实现更倾向于响应中断。在这种情况下，实现必须确保信号被重定向到另一个等待线程（如果有）。


then InterruptedException is thrown and the current thread's interrupted status is cleared. It is not specified, in the first case, whether or not the test for interruption occurs before the lock is released.
