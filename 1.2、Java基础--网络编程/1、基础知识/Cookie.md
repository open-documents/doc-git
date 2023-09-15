
# 什么是cookie?

在web开发中 cookie的含义是`储存在用户本地终端上的数据` 类型为`小型文本型文件`,用Java来说就相当于只支持String类型。

# 属性
| 属性     | 描述                                       |
| -------- | ------------------------------------------ |
| `name`   | cookie的名称,类似于map键值对的key.         |
| `value`  | cookie对应名称的值,类似于map键值对的value. |
| `maxAge` | cookie的最大存活时间.                      |
| `domain` | 可以访问该cookie的站点                     |
| `path`   |定义web站点可以访问该cookie的目录 |

# 写入cookie

(1)声明cookie
```java
Cookie cookie = new Cookie(String name, String value);
```
(2)设置cookie最大存活时间
```java
cookie.setMaxAge(int maxAge);
```
(3)设置可以访问该cookie的站点
```java
cookie.setDomain("站点地址,如:http://localhost:8080");
```
(4)设置站点可以访问该cookie的目录
```java
cookie.setPath("/");
```
(5)使用response添加cookie
```java
response.addCookie(cookie);
```

# 读取cookie

读取cookie需要用到request.

(1)利用request取出所有cookie
```java
Cookie[] cookies = request.getCookies();
```
(2) 遍历cookie数组,取出我们需要的cookie.
假定我们现在要取出name为"LwinnerG"的cookie
```java
for (int i = 0; i < cookies.length; i++) {
    if (cookies[i].getName().equals("LwinnerG")) {
	    retValue = cookies[i].getValue();
	    break;
    }
}
```

# 删除cookie

删除cookie其实就是给指定name的cookie写入空值.然后最大存活时间设为0  
所以还是利用response.

(1)声明Cookie对象
```java
Cookie cookie = new Cookie("要删除的cookie的name", "");
```

(2)设置cookie的最大存活时间
```java
cookie.setMaxAge(0);
```

(3)加入cookie
```java
response.addCookie(cookie);
```
