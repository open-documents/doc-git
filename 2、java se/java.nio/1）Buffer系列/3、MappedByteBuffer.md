MappedByteBuffer通过FileChannel.map()来创建，代表一个文件内容和堆外内存之间的映射。

MappedByteBuffer是一个抽象类，它的实现类为DirectByteBuffer和DirectByteBufferR。

# 新增方法

1）load()

参数：
- 无
功能：将文件

