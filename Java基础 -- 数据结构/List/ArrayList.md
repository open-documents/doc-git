
ArrayList是以数组的方式实现链表，其中使用数组来存储数据。
```java
transient Object[] elementData;
```

）ArrayList中都有哪些属性，都起什么作用？


）`new ArrayList<?>()` 和 `new ArrayList<?>(0)` 有什么区别？

两者都创建一个初始大小为0的数组链表，但是两者初始大小为0的数组指向不同。

`new ArrayList<?>(0)`：初始大小为0的数组指向 EMPTY_ELEMENTDATA。
```java
private static final Object[] EMPTY_ELEMENTDATA = {};

public ArrayList(int initialCapacity) {
	if (initialCapacity > 0) {  
	    // ...
	} 
	else if (initialCapacity == 0) {  
	    this.elementData = EMPTY_ELEMENTDATA;  
	} 
	else {  
	    // ...
	}
}
```
`new ArrayList<?>()`：初始大小为0的数组指向 DEFAULTCAPACITY_EMPTY_ELEMENTDATA
```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() {  
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
}
```


