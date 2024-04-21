# 内存屏障

StoreStore屏障

```java

```

```java
static a = 1;
static volatile b = 2;
// Thread A
a=2;   // b=3;
b=3;   // a=2;

// Thread B
if(b == 3){
	a = 3;
}
```

```java
static a = 1;
static volatile b = 2;
// Thread A
b=3;   // a=2;
if(a == 1){
	...
}

// Thread B
if(b == 3){
	int i = a;
}
```

LoadLoad

```java
int a=1;
volatile b=2;

// Thread A
if(b==2){
	int c=a;
}
// Thread B
b=3;
```


### happens-before 概述

上面我们讲述了重排序原则，为了提高处理速度， JVM 会对代码进行编译优化，也就是指令重排序优化，但是并发编程下指令重排序也会带来一些安全隐患：如**指令重排序导致的多个线程操作之间的不可见性**。为了理解 JMM 提供的内存可见性保证，让程序员再去学习复杂的重排序规则以及这些规则的具体实现，那么程序员的负担就太重了，严重影响了并发编程的效率。所以从 JDK5 开始，提出了 happens-before 的概念，通过这个概念来阐述操作之间的内存可见性。

即定义两个操作之间的happens-before关系：如果一个操作A及该操作A之前的操作对另一个操作B及这个操作B以后的操作可见，那么称A happens-before B。

<font color=44cf57>如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在 happens-before 关系</font>。


这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。

java程序提供保证happens-before 关系的规则如下：

首先必须声明，happens-before 规则不是描述实际操作的先后顺序，而是用来描述可见性的一种规则。

1）程序顺序规则：单线程内的happens-before关系由程序顺序规则保证。

```java
int a=..;  // 或  int a=..;   或  c=..;
c=..;      //     b=a;           int a=..;
b=a;       //     c=..;          b=a;     
```

在上面的栗子中，由业务需求需要，需要满足a happens-before b，而c和a、b之间不需要满足happens-before。因为这里是在单线程内，要满足a happens-before b，只需要利用程序顺序规则。而c没有要求，所以不需要利用程序顺序规则，因此可以写在任何地方。

2）监视器锁规则：对一个监视器锁的解锁，happens-before 于随后对这个监视器锁的加锁。  

3）`volatile` 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。  

```java
static a = 1;
static volatile b = 2;
// Thread A
a=2;
b=3;

// Thread B
System.out.println("a="+a);
if(b == 3){
	System.out.println("a="+a);
}else{
	System.out.println("b!=3");
}
```
这里再次强化happens-before的定义，happens-before并不是操作执行顺序的上约束规则，而是操作导致的可见性的一种约束规则，`b=3` happens-before `b==3`不是说明volatile有线程间同步的功能，而是说明如果（此处用如果是因为无法保证



-   传递性：如果 A happens-before B，且 B happens-before C，那么 A happens-before C。  
    
-   start() 规则：Thread.start() 的调用会 happens-before 于启动线程里面的动作。  
    
-   join() 规则：Thread 中的所有动作都 happens-before 于其他线程从 Thread.join() 中成功返回。



修改：

通过这个概念来阐述操作之间的内存可见性。-> 通过这个概念来阐述操作导致的内存可见性。