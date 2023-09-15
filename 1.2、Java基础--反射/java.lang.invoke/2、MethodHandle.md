# 一、获得Constructor句柄

通过MethodHandles.LoopUp实例方法 `findConstructor(Class<?> refc, MethodType type)`获得。

**参数：**
- refc：Class类型，Constructor方法所属的类。
- type：MethodType类型，方法签名。MethodType实例的返回值类型必须是`void`，这和JVM中Constructor方法的描述符一致。

**注意事项：**
- Constructor和它所有的参数类型必须对lookup对象可见。
- 当Constructor句柄被调用时，如果Constructor的类未初始化，会先对其进行初始化。

**栗子：**

```java
LookUp lookup = MethodHandles.lookup();
MethodType mt = MethodType.methodType(void.class,Collection.class)
MethodHandle MH_newArrayList = lookup.findConstructor(ArrayList.class,mt);
MH_newArrayList.invokeExact(Arrays.asList("x","y"));
```

# 获得Getter句柄


# 获得Setter句柄




