
# native方法

|&nbsp;&nbsp;&nbsp;&nbsp;native方法&nbsp;&nbsp;&nbsp;&nbsp;    |         描述         |
|:----------:|:--------------------:|
| isArray()  | 判断该类是否是数组。 |
|            |                      |

# getPackageName()
```java

```
# getModule()

如果Class是数组类型，返回元素类型所在的Module。
如果Class是primitive或void类型，返回 java.base module。
如果Class所在的是unnamed模组，返回这个类的ClassLoader.getUnnamedModule()。
```java
private transient Module module;  // set by VM  
public Module getModule() {  
    return module;  
}
```



# getComponentType()
```java
private final Class<?> componentType;
public Class<?> getComponentType() {  
    if (isArray()) {  
        return componentType;  
    } else {  
        return null;  
    }  
}
```


# getEnclosingClass()
```
```
# getDeclaringClass()

