
>参考文档 1：《深入理解Java虚拟机 第3版》 第2章（后文简称《理解》）[1]
>参考文档 2：https://docs.oracle.com/javase/specs/jvms/se18/html/jvms-2.html#jvms-2.5（后文简称《规范》）[2]


---
堆（Heap，为了与数据结构堆区分，后文记作JVM堆）

JVM堆用来存储所有类实例对象（即new出来的对象）、所有数组。JVM堆为所有线程共有[1]。

JVM堆的内存大小可以是初始指定大小的固定大小，也可以初始指定最小、最大大小的可变大小。如果[1]

---
方法区（Method Area）



运行时常量池（Run-Time Constant Pool）