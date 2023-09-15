	参考文档:P943
	
ServerResponse提供了对HTTPResponse的访问，由于它是不可变的，所以可以使用build的方法来创建它。可以使用builder设置响应状态，添加响应头，或者响应体。


# 例子

1）状态码200，设置回复头，设置回复体。
```java
Person person = ... 
ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(person);
```
2）状态码201，设置回复头。
```java
URI location = ... 
ServerResponse.created(location).build();
```

# 异步设置（不懂）

	参考文档:P944
