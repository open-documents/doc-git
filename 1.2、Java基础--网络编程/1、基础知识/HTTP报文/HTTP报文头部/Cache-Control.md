**`Cache-Control`** 通用消息头字段，被用于在 http 请求和响应中，通过指定指令来实现缓存机制。缓存指令是单向的，这意味着在请求中设置的指令，不一定被包含在响应中。

# 语法

指令格式具有以下有效规则：
- 不区分大小写，但建议使用小写。
- 多个指令以逗号分隔。
- 具有可选参数，可以用令牌或者带引号的字符串语法。

# 缓存请求指令

客户端可以在 HTTP请求中使用的标准 Cache-Control 指令。
```text
Cache-Control: max-age=<seconds>
Cache-Control: max-stale[=<seconds>]
Cache-Control: min-fresh=<seconds>
Cache-control: no-cache
Cache-control: no-store
Cache-control: no-transform
Cache-control: only-if-cached
```

# 缓存响应指令

服务器可以在HTTP响应中使用的标准 Cache-Control 指令。
```text
Cache-control: must-revalidate
Cache-control: no-cache
Cache-control: no-store
Cache-control: no-transform
Cache-control: public
Cache-control: private
Cache-control: proxy-revalidate
Cache-Control: max-age=<seconds>
Cache-control: s-maxage=<seconds>
```

# 指令
## 可缓存性

1）public
表明响应可以被任何对象（包括：发送请求的客户端，代理服务器，等等）缓存，即使是通常不可缓存的内容。（例如：1.该响应没有max-age指令或Expires消息头；2. 该响应对应的请求方法是 POST 。）

2）private
表明响应只能被单个用户缓存，不能作为共享缓存（即代理服务器不能缓存它）。私有缓存可以缓存响应内容，比如：对应用户的本地浏览器。

3）no-cache
在发布缓存副本之前，强制要求缓存把请求提交给原始服务器进行验证 (协商缓存验证)。

4）no-store
缓存不应存储有关客户端请求或服务器响应的任何内容，即不使用任何缓存。