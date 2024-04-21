
>参考文档：JDK18.0.1源码

JDK提供<span style="color:#00E0FF">CountDownLatch</span>、CyclicBarrier、Phaser 3种同步工具，下面介绍<span style="color:#00E0FF">CountDownLatch</span>。

<span style="color:#00E0FF">CountDownLatch</span>允许线程等待其它线程完成工作后继续执行。需要注意的是<span style="color:#00E0FF">CountDownLatch</span>是<span style="color:#fb463c">一次性</span>的，即<span style="color:#fb463c">无法</span>将<span style="color:#00E0FF">CountDownLatch</span>重置(reset)到初始状态。

<span style="color:#00E0FF">CountDownLatch</span>使用一个数字count来进行初始化，执行await()的线程在count减少到0后继续执行，其它线程通过countDown()来减小count。

# 使用情境

下面通过案例，来体会<span style="color:#00E0FF">CountDownLatch</span>的作用。

```java
class Driver {
	// ...    
	void main() throws InterruptedException {      
		CountDownLatch startSignal = new CountDownLatch(1);      
		CountDownLatch doneSignal = new CountDownLatch(N);        
		for (int i = 0; i < N; ++i) {
			// create and start threads        
			new Thread(new Worker(startSignal, doneSignal)).start();
		}
		doSomethingElse1();           // don't let run yet
        startSignal.countDown();      // let all threads proceed      
        doSomethingElse2();
        doneSignal.await();           // wait for all to finish 
	} 
}    

class Worker implements Runnable {
	private final CountDownLatch startSignal; 
	private final CountDownLatch doneSignal;  
	Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {   
		this.startSignal = startSignal;  
		this.doneSignal = doneSignal; 
	}   
	public void run() {  
		try {      
			startSignal.await();    
			doWork();    
			doneSignal.countDown();   
		} 
		catch (InterruptedException ex) {}
		return;  
	}    
	void doWork() { ... } 
}
```

startSignal是CountDownLatch(1)，用于主线程完成部分工作doSomethingElse1()后，让其它线程开始执行。
doneSignal是CountDownLatch(N)，用于其它线程完成自身工作后，让主线程退出。