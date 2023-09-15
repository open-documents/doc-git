# JVM栈帧

## 一、概述

### 栈帧位置

JVM 执行 Java 程序时需要装载各种数据到内存中，不同的数据存放在不同的内存区中（逻辑上），这些数据内存区称作[运行时数据区](C:\Users\XiangCXiao\Desktop\Typora\JVM\Java-JVM 运行时内存结构（Run-Time Data Areas）.md)

其中 JVM Stack（Stack 或虚拟机栈、线程栈、栈）中存放的就是 Stack Frame（Frame 或栈帧、方法栈）。

### 对应关系

一个线程对应一个 JVM Stack。JVM Stack 中包含一组 Stack Frame。线程每调用一个方法就对应着 JVM Stack 中 Stack Frame 的入栈，方法执行完毕或者异常终止对应着出栈（销毁）。

当 JVM 调用一个 Java 方法时，它从对应类的类型信息中得到此方法的局部变量区和操作数栈的大小，并据此分配栈帧内存，然后压入 JVM 栈中。

在活动线程中，只有位于栈顶的栈帧才是有效的，称为当前栈帧，与这个栈帧相关联的方法称为当前方法。

<img src="C:\Users\XiangCXiao\AppData\Roaming\Typora\typora-user-images\image-20210512140741504.png" alt="image-20210512140741504" style="zoom: 50%;" />

https://www.cnblogs.com/jhxxb/p/11001238.html