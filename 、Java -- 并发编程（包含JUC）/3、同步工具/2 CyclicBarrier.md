
>参考文档：JDK18.0.1源码

JDK提供CountDownLatch、<span style="color:#00E0FF">CyclicBarrier</span>、Phaser 3种同步工具，下面介绍<span style="color:#00E0FF">CyclicBarrier</span>。



# 使用情境

下面通过案例，来体会<span style="color:#00E0FF">CyclicBarrier</span>的作用。

```java
class Solver {
	final int N; 
	final float[][] data; 
	final CyclicBarrier barrier;    

	public Solver(float[][] matrix) { 
		data = matrix;   
		N = matrix.length; 
		Runnable barrierAction = () -> mergeRows(...);  
		barrier = new CyclicBarrier(N, barrierAction);   
		List<Thread> threads = new ArrayList<>(N);   
		for (int i = 0; i < N; i++) {    
			Thread thread = new Thread(new Worker(i));   
			threads.add(thread);  
			thread.start();   
		}  
		
		// wait until done  
		for (Thread thread : threads)    
			try {    
				thread.join();  
			} 
			catch (InterruptedException ex) { }  
		}
	}
}

class Worker implements Runnable {   
	int myRow;   
	Worker(int row) { myRow = row; }  
	public void run() {     
		while (!done()) {    
			processRow(myRow);    
			try {     
				barrier.await();  
			} 
			catch (InterruptedException ex) {      
				return;   
			} 
			catch (BrokenBarrierException ex) {   
				return;   
			}     
		}  
	}  
}     
```