锁和Synchronizer暂时没有太大的含义差别。

这个类是所有互斥锁的基类。

它表示一个可能被一个线程单独占有的Synchronizer。

## 属性及属性方法

```java
private transient Thread exclusiveOwnerThread;  // 拥有当前锁的线程

protected final void setExclusiveOwnerThread(Thread thread) {  exclusiveOwnerThread = thread;}
protected final Thread getExclusiveOwnerThread() {  return exclusiveOwnerThread; }
```

