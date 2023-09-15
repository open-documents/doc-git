
参考文档：
- 源码
- https://blog.csdn.net/u022812849/article/details/107513876

ThreadLocal

## get()


```java
public T get() {  
    Thread t = Thread.currentThread();  
    // return t.threadLocals; 
    // 每个线程的threadLocals 初始值为null
    ThreadLocalMap map = getMap(t);  
    if (map != null) {  
        ThreadLocalMap.Entry e = map.getEntry(this);  
        if (e != null) {  
            @SuppressWarnings("unchecked")  
            T result = (T)e.value;  
            return result;  
        }  
    }  
    return setInitialValue();  
}
```


# 内存泄漏问题


参考文档：https://blog.csdn.net/u022812849/article/details/107513876

内存泄漏：是指本应该被GC回收的无用对象没有被回收，导致内存空间的浪费，当内存泄露严重时会导致内存溢出。Java内存泄露的根本原因是：长生命周期的对象持有短生命周期对象的引用，尽管短生命周期对象已经不再需要，但是因为长生命周期对象持有它的引用而导致不能被GC回收。

内存溢出：就是OOM（OutOfMemoryError）异常，简单理解就是内存不够了，通常发生在程序申请的内存超出了JVM中可用内存的大小，就会抛出OOM异常。在JVM内存区域中，除了程序计数器外其他的内存区域都有可能抛出OOM异常。

ThreadLocal很好地解决了线程之间需要数据隔离的问题，同时也引入了另一个问题，在应用程序中通常会使用线程池来管理线程，那么线程的生命周期与应用程序的生命周期基本保持一致，如果线程的数量很多，随着程序的运行，时间的推移，ThreadLocal类型的变量会越来越多，将会占用非常大的内存空间，从而产生内存泄漏，如果这些对象一直不被释放的话，可能会导致内存溢出。

## ThreadLocal的OOM演示

下面的代码中开启了十个线程，每个线程申请10M的内存，而给堆只分配了100M，这样到后面的线程申请内存时就会抛出OOM异常。

```java
ThreadLocal<byte[]> threadLocal = new ThreadLocal<>();
CyclicBarrier cyclicBarrier = new CyclicBarrier(12);
for (int i = 0; i < 10; i++) {
    new Thread(()->{
        threadLocal.set(new byte[1024 * 1024 * 10]); // java.lang.OutOfMemoryError: Java heap space
        // threadLocal.remove();
        try {
            cyclicBarrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }).start();
}
cyclicBarrier.await(); // 保证main线程不退出
```


