
# 实现类

## InputStreamReader

InputStreamReader是字节流到字符流的桥梁：读取字节，然后使用特定的charset将这些字节转换成字符。charset可以通过名称指定，可以显示指定，也可以使用平台默认的字符集（见构造函数）。

每次调用read()方法从字节流中读取一个或多个字节时，为了提高将字节转换成字符的效率，可以提前读入比满足这次read()所需字节更多的字节数。

**1）构造函数**
```java
public InputStreamReader(InputStream in);                      // 使用平台默认字符集
public InputStreamReader(InputStream in, String charsetName);  // 通过名称指定字符集
public InputStreamReader(InputStream in, Charset cs);          // 显式地指定字符集
```