
JUC提供3种同步工具：
- CountDownLatch
- CyclicBarrier
- Phaser（本文介绍）

Phaser是一个可以反复使用的同步工具。

总体而言，Phaser有以下特征：
- Phaser记录了能够容纳的同步的成员（party）数量，这个数量与CyclicBarrier不同，可以动态变化。1、通过Phaser构造器方式指定，2、通过register()、bulkRegister()来增加，3、通过arriveAndDeregister()来减少。
- 随着参与同步的成员的arrive，arrive的数量逐渐增加，当 arrive的成员数量与 Phaser中能够容纳的同步的数量相同时，Phaser进入（advance）下一阶段，同时调用onAdvance()方法。
- Phaser有一个所处的phase，当Phaser进入(advance)下一阶段时，Phaser所处的phase增加1，这个数从0开始逐渐增大到 Integer.MAX_VALUE，然后继续从0开始增加。

# 注册（Registration）

与CyclicBarrier不同，Phaser容纳的能够同步的成员（party）数量是可以动态变化的。可以在任何时刻通过下面的方式来设定（改变）：
- 第一次创建Phaser实例时，使用构造器指定初始数量的方式来增加Phaser可以同步的成员数量。
- 使用 register()，bulkRegister 来增加。
- 使用 arriveAndDeregister() 来减少Phaser容纳的同步的成员数量。

下面对这些方法做介绍。

1）register()
```java
public int register() {  return doRegister(1);  }
```

使Phaser能容纳的同步的成员（party）数量加1，返回注册时Phaser所处的phase，如果返回负数，表明该Phaser已经处于terminated状态。

注意事项：
- 如果onAdvance()正在发生，那么该方法在返回前可能会等待onAdvance()完成。
- If this phaser has a parent, and this phaser previously had no registered parties, this child phaser is also registered with its parent. 
- 如果该Phaser已经处于terminated，那么调用该方法不会产生任何影响，会直接返回一个负数。

2）bulkRegister(int)
```java
public int bulkRegister(int parties) {  
    if (parties < 0)  
        throw new IllegalArgumentException();  
    if (parties == 0)  
        return getPhase();  
    return doRegister(parties);  
}
```

和register()类似，使得Phaser能容纳的同步的成员（party）数量加指定数量（参数parties）。

3）

# arrive



# onAdvance

