
Channel代表一个到实体的连接，这个实体可以是一个<font color=44cf57>硬件设备</font>、一个<font color=44cf57>文件</font>、一个<font color=44cf57>网络套接字</font>甚至可以是一个<font color=44cf57>能进行I/O操作的程序组件</font>。

Channel有如下扩展子接口：
- ByteChannel。ByteChannel没有增加功能，只是 ReadableByteChannel和WritableByteChannel的一个合成接口。ByteChannel有一个增强子接口SeekableByteChannel。
- ReadableByteChannel。ReadableByteChannel有扩展子接口ScatteringByteChannel。
- WritableByteChannel。WritableByteChannel有扩展子接口GatheringByteChannel。
- NetworkChannel
- InterruptibleChannel

Channel有一个子抽象类：
- SelectableChannel
-- --
# 1）Channel接口定义

下面是Channel接口的定义：
```java
public interface Channel extends Closeable{
	public boolean isOpen();
	public void close() throws IOException;
}
```

# 2）读通道

读通道是从通道连接的实体中读入若干字节，然后写入到若干Buffer中。

## 2.1）ReadableByteChannel

ReadableByteChannel只有一个方法`read(ByteBuffer) : int`

read(ByteBuffer)方法表示从channel连接的实体中读取若干字节到ByteBuffer中。

下面是ReadableByteChannel的定义：
```java

```
## 2.2）ScatteringByteChannel

ScatteringByteChannel是ReadableByteChannel的扩展接口。下面是ScatteringByteChannel的定义：
```java
public interface ScatteringByteChannel extends ReadableByteChannel{
	public long read(ByteBuffer[] dsts, int offset, int length) throws IOException;
	public long read(ByteBuffer[] dsts) throws IOException;
}
```

方法：
- `read(ByteBuffer[] dsts, int offset, int length)`：从该通道连接的实体中读取若干字节，然后写入到下标offset ~ length-1的Buffer中。
- `read(ByteBuffer[] dsts)`：等价于`read(dsts, 0, dsts.length)`。

# 3）写通道

写通道是从若干Buffer中读取若干字节，然后写入到通道连接的实体中。
## 3.1）WritableByteChannel

WritableByteChannel只有一个方法`write(ByteBuffer) : int`

write(ByteBuffer)方法：
- 尝试将ByteBuffer的剩余(remaining()方法返回值)若干字节写入到该channel中。
- 非阻塞模式的socketChannel不能写入超过socketBuffer剩余字节的字节数到channel中。
- 当一个线程调用了write()方法向channel写入，其他线程再次调用时会被阻塞，一直到第一个写入操作完成。

下面是WritableByteChannel的定义：
```java
public interface WritableByteChannel extends Channel{
	public int write(ByteBuffer src) throws IOException;
}
```

## 3.2）GatheringByteChannel

GatheringByteChannel是WritableByteChannel的扩展接口。下面是GatheringByteChannel的定义：
```java
public interface GatheringByteChannel extends WritableByteChannel{
	public long write(ByteBuffer[] srcs, int offset, int length) throws IOException;
	public long write(ByteBuffer[] srcs) throws IOException;
}
```

方法：
- `write(ByteBuffer[] srcs, int offset, int length)`：从下标 `offset ~ offset+length-1` 的ByteBuffer数组中读取若干字节，然后写入到该通道连接的实体中。
- `write(ByteBuffer[] srcs)`：等价于`write(srcs, 0, srcs.length)`。

# 4）SeekableByteChannel

SeekableByteChannel的定义如下：
```java
public interface SeekableByteChannel extends ByteChannel{
	long position() throws IOException;
	SeekableByteChannel position(long newPosition) throws IOException;
	long size() throws IOException;
	SeekableByteChannel truncate(long size) throws IOException;
}
```
方法：
- `position()`：获取当前通道连接实体的数据的操作位置。
- `position(long newPosition)`：修改当前通道连接实体的数据的操作位置。
- `size()`：获取当前通道连接实体的数据的大小。
- `truncate(long size)`：对当前通道连接实体的数据进行截断，如果。
# 5）NetworkChannel

NetworkChannel顾名思义，代表一个连接网络套接字的通道。

下面是NetworkChannel的定义：
```java
public interface NetworkChannel extends Channel{
	NetworkChannel bind(SocketAddress local) throws IOException;
	SocketAddress getLocalAddress() throws IOException;
	<T> NetworkChannel setOption(SocketOption<T> name, T value) throws IOException;
	<T> T getOption(SocketOption<T> name) throws IOException;
	Set<SocketOption<?>> supportedOptions();
}
```

- bind()：绑定该通道连接到的socket到指定的SocketAddress。

# 5）InterruptibleChannel

InterruptibleChannel顾名思义，代表一个可以<font color=44cf57>异步关闭</font>和<font color=44cf57>异步打断</font>的通道。

异步关闭：
- 一个线程通过该通道进行读写被阻塞时，另一个线程可以通过close()来关闭这个通道，但是被阻塞线程会收到AsynchronousCloseException。
异步打断：
- 一个线程通过该通道进行读写被阻塞时，另一个线程可以通过调用被阻塞线程的interrupt()方法来打断这个通道，但是被阻塞线程会收到 ClosedByInterruptException，并且被阻塞线程的interrupt status会被设置为true。
- 如果一个线程的interrupt status已经为true，然后该线程还调用该通道的读写操作，那么该通道会被立刻关闭，并且该线程也会收到ClosedByInterruptException异常，interrupt status保持为true。
- 一个通道当且仅当实现了接口时才能被异步关闭和打断。可以通过instanceof来进行测试。

下面是InterruptibleChannel的定义：
```java
public interface InterruptibleChannel extends Channel{
	public void close() throws IOException;
}
```

# 6）SelectableChannel

SelectableChannel是一个抽象类。除了自身的功能，还继承了InterruptibleChannel，也就是能被异步打断和关闭。

SelectableChannel顾名思义，代表一个可以通过选择器进行多路复用的信道。

下面是SelectableChannel的定义：
```java
public abstract class SelectableChannel extends AbstractInterruptibleChannel implements Channel{
    public final SelectionKey register(Selector sel, int ops) throws ClosedChannelException{...}
    public abstract SelectionKey register(Selector sel, int ops, Object att) throws ClosedChannelException;
}
```

方法：
- register()：
