# 获得VarHandle实例

**1）获得对象属性的VarHandle实例**

`findVarHandle(Class<?> recv, String name, Class<?> type)`

返回的VarHandle：variable type 是 type， coordinate types是recv。



访问限制：

- `final` 属性，不支持 `write` 、`atomic update`、`numeric atomic update`、`bitwise atomic update`
- `boolean` 不支持 `numeric atomic update`
- `float`、`double`不支持 `bitwise atomic update`
- reference只支持 `write` 、`atomic update`


**2）获得类属性(static field)的VarHandle实例**

`findStaticVarHandle(Class<?> decl, String name, Class<?> type) : VarHandle`

返回的VarHandle：variable type 是 type，无coordinate types。

