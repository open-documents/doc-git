Buffer是一个抽象类。

Buffer代表一个缓存数组，可向其中写入数据或从其中读取数据。

初始状态没有数据，先通过channel向其中写入数据，然后调用flip()方法，转为读模式，然后通过channel从其中读取数据。

1）Buffer有如下所有抽象子类：
- <font color=44cf57>Xxx</font>Buffer
- <font color=44cf57>Xxx</font>BufferAs<font color=44cf57>Xxx</font>Buffer<font color=44cf57>A</font>
Xxx：表示Byte、Char、Short、Int、Long、Float、Double的其中一个。
A：表示B、L、RB、RL的其中一个。

2）ByteBuffer有如下抽象子类：
- MappedByteBuffer。有实现类DirectXxxB
- HeapXxxBufferE

B：表示C、D中的一个。
C：当Xxx为Byte时，表示可选、R（可读）的其中一个。
D：当Xxx为非Byte时，表示RS、RU、S、U的其中一个。
E：表示可选和R的其中一个。
# 1）Buffer四个属性：
- mark
- position
- limit
- capacity

一定满足的关系：mark <= position <= limit <= capacity

能够访问的属性：position、limit、capacity
能够设置的属性：position、limit


# 2）重要方法

## flip()

用于读写翻转。

在一系列通道读或put操作后，需要调用flip()来从写变成读。
```java
buf.put(magic);    // Prepend header
in.read(buf);      // Read data into rest of buffer
buf.flip();        // Flip buffer
out.write(buf);    // Write header + data to channel
```


## clear()

恢复Buffer至初始状态。即position = 0，mark = -1，limit = capacity。

一般用在全部读取buffer数据之前。
```java
buf.clear();     // Prepare buffer for reading  
in.read(buf);    // Read data</pre></blockquote>
```

# 3）用途

## 从指定位置多次重复读

首先mark()，标记需要重复读的起点位置。
读取数据，position增大。
然后reset()，position恢复至mark()位置。
重复读取数据。
...


