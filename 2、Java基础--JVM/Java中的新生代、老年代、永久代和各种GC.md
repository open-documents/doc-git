# Java中的新生代、老年代、永久代和各种GC



## 1 前言

JVM中的堆，一般分为三大部分：新生代、老年代、永久代

![image-20210512134944824](C:\Users\XiangCXiao\AppData\Roaming\Typora\typora-user-images\image-20210512134944824.png)



## **2 新生代**

主要是用来存放新生的对象。一般占据堆的1/3空间。由于频繁创建对象，所以新生代会频繁触发**MinorGC**进行垃圾回收。

新生代又分为 Eden区、ServivorFrom、ServivorTo三个区。

- Eden区：Java新对象的出生地（如果新创建的对象占用内存很大，则直接分配到老年代）。当Eden区内存不够的时候就会触发MinorGC，对新生代区进行一次垃圾回收。
- ServivorTo：保留了一次MinorGC过程中的幸存者。
- ServivorFrom：上一次GC的幸存者，作为这一次GC的被扫描者。

当JVM无法为新建对象分配内存空间的时候(Eden满了)，Minor GC被触发。因此新生代空间占用率越高，Minor GC越频繁。

MinorGC的过程：采用复制算法。

1. 首先，把Eden和ServivorFrom区域中存活的对象复制到ServicorTo区域（如果有对象的年龄以及达到了老年的标准，一般是15，则赋值到老年代区）
2. 同时把这些对象的年龄+1（如果ServicorTo不够位置了就放到老年区）
3. 然后，清空Eden和ServicorFrom中的对象；最后，ServicorTo和ServicorFrom互换，原ServicorTo成为下一次GC时的ServicorFrom区。

![image-20210512135034759](C:\Users\XiangCXiao\AppData\Roaming\Typora\typora-user-images\image-20210512135034759.png)



## **3 老年代**

老年代的对象比较稳定，所以MajorGC不会频繁执行。

在进行MajorGC前一般都先进行了一次MinorGC，使得有新生代的对象晋身入老年代，导致空间不够用时才触发。当无法找到足够大的连续空间分配给新创建的较大对象时也会提前触发一次MajorGC进行垃圾回收腾出空间。

MajorGC采用标记—清除算法：

1. 首先扫描一次所有老年代，标记出存活的对象
2. 然后回收没有标记的对象。

MajorGC的耗时比较长，因为要扫描再回收。MajorGC会产生内存碎片，为了减少内存损耗，我们一般需要进行合并或者标记出来方便下次直接分配。

当老年代也满了装不下的时候，就会抛出OOM（Out of Memory）异常。



## **4 永久代**

指内存的永久保存区域，主要存放Class和Meta（元数据）的信息。

Class在被加载的时候被放入永久区域。它和和存放实例的区域不同，GC不会在主程序运行期对永久区域进行清理。所以这也导致了永久代的区域会随着加载的Class的增多而胀满，最终抛出OOM异常。

在Java8中，永久代已经被移除，被一个称为“元数据区”（元空间）的区域所取代。

元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制。类的元数据放入 native memory, 字符串池和类的静态变量放入java堆中. 这样可以加载多少类的元数据就不再由MaxPermSize控制, 而由系统的实际可用空间来控制。



## 5 GC

总类：Minor GC、Major GC、Full GC

- Minor GC

当年轻代满时就会触发Minor GC，这里的年轻代满指的是Eden代满，Survivor满不会引发GC。通过复制算法 ,回收垃圾。复制算法不会产生内存碎片。

 **复制算法将内存划分为两个区间，在任意时间点，所有动态分配的对象都只能分配在其中一个区间（称为活动区间），而另外一个区间（称为空闲区间）则是空闲的**。

<img src="C:\Users\XiangCXiao\AppData\Roaming\Typora\typora-user-images\image-20210512140230041.png" alt="image-20210512140230041" style="zoom:50%;" />

当有效内存空间耗尽时，JVM将暂停程序运行，开启复制算法GC线程。**接下来GC线程会将活动区间内的存活对象，全部复制到空闲区间，且严格按照内存地址依次排列，与此同时，GC线程将更新存活对象的内存引用地址指向新的内存地址**。

<img src="C:\Users\XiangCXiao\AppData\Roaming\Typora\typora-user-images\image-20210512140245055.png" alt="image-20210512140245055" style="zoom:50%;" />

此时，空闲区间已经与活动区间交换，而垃圾对象现在已经全部留在了原来的活动区间，也就是现在的空闲区间。事实上，在活动区间转换为空间区间的同时，垃圾对象已经被一次性全部回收。

- Major GC

Major GC又称为Full GC。当年老代空间不够用的时候，虚拟机会使用“标记—清除”或者“标记—整理”算法清理出连续的内存空间，分配对象使用。

虚拟机给每个对象定义了一个对象年龄（Age）计数器。如果对象在 Eden 出生并经过第一次 Minor GC 后仍然存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间中，并将对象年龄设为 1。对象在 Survivor 区中每熬过一次 Minor GC，年龄就增加 1 岁，当它的年龄增加到一定程度（默认为 15 岁）时，就会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数 **-XX:MaxTenuringThreshold** (阈值)来设置。



