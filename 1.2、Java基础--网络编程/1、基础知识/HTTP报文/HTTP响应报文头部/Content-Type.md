	参考文档:https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/

`Content-Type` 实体头部用于指示资源的 MIME 类型 media type。

media type如下：
- application/atom+xml
- application/cbor
- application/x-www-form-urlencoded
- application/octet-stream
- application/json
- application/xml
- application/pdf
- application/x-ndjson
- application/graphql+json
- application/graphql-response+json
- application/problem+json
- application/problem+xml
- application/rss+xml
- application/x-protobuf
- application/stream+json
- application/xhtml+xml
- image/gif
- image/jpeg
- image/png
- multipart/form-data
- multipart/mixed
- multipart/related
- text/event-stream
- text/html
- text/markdown
- text/plain
- text/xml

在响应中，Content-Type 标头告诉客户端从服务端实际返回内容的内容类型。浏览器会在某些情况下进行 MIME 查找，并不一定遵循此标题的值。为了防止这种行为，可以将标题 `X-Content-Type-Options`设置为 **nosniff**。

在请求中 (如`POST`或`PUT`)，客户端告诉服务器实际发送的数据类型。

```text
Content-Type: text/html; charset=utf-8
Content-Type: multipart/form-data; boundary=something
```

media-type：资源或数据的MIME type。
charset：字符编码标准。
boundary：对于多部分实体，boundary 是必需的，其包括来自一组字符的 1 到 70 个字符，已知通过电子邮件网关是非常健壮的，而不是以空白结尾。它用于封装消息的多个部分的边界。

## 例子

在通过 HTML form 提交生成的`POST`请求中，请求头的 Content-Type 由`<form>`元素上的 enctype 属性指定。
```xml
<form action="/" method="post" enctype="multipart/form-data">
	<input type="text" name="description" value="some text">
	<input type="file" name="myFile">
	<button type="submit">Submit</button>
</form>
```