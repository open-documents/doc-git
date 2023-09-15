	参考文档:P942

ServerRequest和ServerResponse提供对JDK8中的HTTPRequest和HTTPResponse的友好访问方式。两者都不能修改。


通过ServerRequest能够访问：
- HTTP请求方法。
- URI。
- 请求参数。
- 请求体。

# 访问请求体

1）将请求体转变为String。
```java
String string = request.body(String.class);
```
2）将请求体转变为`List<Person>`，Person通过解码被序列化的JSON或XML得到。
```java
List<Person> people = request.body(new ParameterizedTypeReference<List<Person>>() {});
```


# 访问参数

```java
MultiValueMap<String, String> params = request.params();
```

