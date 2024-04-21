
>参考：JDK 18.0.1 ArrayList源码

<span style="color:#00E0FF">ArrayList</span>是以数组的方式实现列表，使用数组来存储数据。
```java
transient Object[] elementData;
```

<span style="color:#fb463c">）问题：为什么存储数据的数组元素类型是Object？</span>


## 初始化

<span style="color:#00E0FF">ArrayList</span>有3种初始化方式：
- `ArrayList()`：无参构造，默认初始容量为0。
- `ArrayList(int initialCapacity)`：构造指定初始容量的<span style="color:#00E0FF">ArrayList</span>。
- `ArrayList(Collection<? extends E> c)`：使用另外一个集合来初始化<span style="color:#00E0FF">ArrayList</span>。

<span style="color:#fb463c">）问题：new ArrayList<?>() 与 new ArrayList<?>(0) 有什么区别？</span>

`new ArrayList<?>()`与`new ArrayList<?>(0)`主要有2个区别：
- `new ArrayList<?>()`在没有第一次添加元素前，确保容量（即ensureCapacity()）不生效。
- `new ArrayList<?>()`在第一次添加元素后，容量直接增长到默认容量（即10），而`new ArrayList<?>(0)`容量变为1。

所以，如果能够保证元素数量不大于10个，可以使用`new ArrayList<?>(0)`，而非`new ArrayList<?>()`来节约少量内存空间。

2个区别的出现，需要知道：虽然两者都创建一个初始大小为0的<span style="color:#00E0FF">ArrayList</span>，但是两者的内存空间不同：
- `new ArrayList<?>(0)`初始大小为0的数组EMPTY_ELEMENTDATA
- `new ArrayList<?>()`初始大小为0的数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA
```java
private static final Object[] EMPTY_ELEMENTDATA = {};                   // ArrayList(0)
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};   // ArrayList()
```

下面介绍为何`new ArrayList<?>()`在没有第一次添加元素前，确保容量（即ensureCapacity()）不生效。
```java
/**
 * 
 */
public void ensureCapacity(int minCapacity) {  
    if (minCapacity > elementData.length  
        && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA   // new ArrayList<?>()初始指向, 第一次添加元素后改变
        && minCapacity <= DEFAULT_CAPACITY)) {  
		// ... 无关代码
    }  
}
```
关于为何`new ArrayList<?>()`在第一次添加元素后，容量直接增长到默认容量（即10），而`new ArrayList<?>(0)`容量变为1，在扩容部分讲解。

## 扩容

向<span style="color:#00E0FF">ArrayList</span>添加元素（`add(E)`），空间不够时，需要扩容，大概流程是：1）分配容量大小足够的内容空间；2）拷贝原始<span style="color:#00E0FF">ArrayList</span>所有元素到新的内存空间中。

---
下面介绍<span style="color:#00E0FF">ArrayList</span>每次扩容大小。

首先，介绍2个阈值：<span style="color:#FFDD00">软最大数组长度 SOFT_MAX_ARRAY_LENGTH</span>（Integer.MAX_VALUE - 8 = 2147483639），<span style="color:#FF5500">数组最大长度</span>（Integer.MAX_VALUE，即2^31-1）

<span style="color:#fb463c">为什么存在软最大数组长度？</span>根据源码介绍，部分JVM实现（如HotSpot）在尝试申请接近<span style="color:#FF5500">数组最大长度</span>大小的数组时，即使有足够的堆运行时内存，也会抛出异常，为了减小这种可能的出现，引入了<span style="color:#FFDD00">软最大数组长度</span>。

<span style="color:#44cf57">每次扩容的大小</span>：
）原始容量计划增长原始容量的一半，如果没有没有超过<span style="color:#FFDD00">软最大数组长度</span>，则扩容原始容量一半大小。（情况1）
）原始容量计划增长原始容量的一半，如果超过了<span style="color:#FFDD00">软最大数组长度</span>，则原始容量计划增长最小增长量（比如1）
- 如果没有超过<span style="color:#FFDD00">软最大数组长度</span>，则扩容到<span style="color:#FFDD00">软最大数组长度</span>。（情况2.1）
- 如果超过了<span style="color:#FFDD00">软最大数组长度</span>，但是没有超过<span style="color:#FF5500">数组最大长度</span>，则扩容最小增长量（比如1）。（情况2.2）
- 如果超过<span style="color:#FF5500">数组最大长度</span>，则抛出OutOfMemoryError异常。（情况2.3）

下面对上面的原理作源码解析，以`add(E)`出现扩容需求为例，下面是需要扩容时的整体调用流程。
```text
add(E) -> add(E, Object[], int) -> grow() -> grow(int minCapacity)
```
忽略无关过程，下面重点介绍grow(int minCapacity)扩容过程，minCapacity表示扩容的最小增长量，在add(E)时，因为只需要添加一个元素，所以minCapacity为1。
```java
/**
 * ArrayList
 */
private Object[] grow(int minCapacity) {  
    int oldCapacity = elementData.length;  
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {  
        int newCapacity = ArraysSupport.newLength(
		    oldCapacity,  
            minCapacity - oldCapacity,  /* minimum growth */  
            oldCapacity >> 1            /* preferred growth */);  
        return elementData = Arrays.copyOf(elementData, newCapacity);  
    } 
    else {  
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];  
    }  
}
```

）首先，上述代码的`else`回答了<span style="color:#44cf57">new ArrayList()在第一次扩容时，容量不会变为1，而是变为10</span>。

）`ArraysSupport.newLength(int, int, int)`即为扩容大小。下面对其作介绍。
```java
/**
 * ArraysSupport
 *
 * Parameters:
 *     oldLength: 旧容量
 *     minGrowth: 最小增长容量, 对于add(E)来说为1
 *     prefGrowth: 预期增长容量, 大小为旧容量的一半
 */
public static int newLength(int oldLength, int minGrowth, int prefGrowth) {  
    int prefLength = oldLength + Math.max(minGrowth, prefGrowth); // might overflow  
    // 情况 1: 原始容量计划增长原始容量的一半, 没有软最大数组长度, 扩容一半
    if (0 < prefLength && prefLength <= SOFT_MAX_ARRAY_LENGTH) {  
        return prefLength;  
    } 
    // 情况 2.1, 2.2, 2.3: 原始容量计划增长原始容量的一半, 超过软最大数组长度
    else {  
        return hugeLength(oldLength, minGrowth);  
    }  
}
```
`else`部分是原始容量计划增长一倍，但是超过了最大容量。
```java
/**
 * ArraysSupport
 */
private static int hugeLength(int oldLength, int minGrowth) {  
    int minLength = oldLength + minGrowth;  
    // 情况 2.3: 原始容量计划增长最小增长量(比如1), 但是超过了数组最大长度, 抛出异常
    if (minLength < 0) { // overflow  
        throw new OutOfMemoryError("Required array length " + oldLength + " + " + minGrowth + " is too large");  
    } 
    // 情况2.1 : 原始容量计划增长最小增长量(比如1), 没有超过软最大数组长度，扩容到软最大数组长度
    else if (minLength <= SOFT_MAX_ARRAY_LENGTH) {  
        return SOFT_MAX_ARRAY_LENGTH;  
    } 
    // 情况2.2 : 原始容量计划增长最小增长量(比如1), 超过软最大数组长度但是没有超过数组最大长度, 扩容最小增长量
    else {  
        return minLength;  
    }  
}
```







