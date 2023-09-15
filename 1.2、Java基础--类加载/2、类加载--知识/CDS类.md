
# 静态方法块

CDS类有下面的静态方法块。
```java
static {  
    isDumpingClassList = isDumpingClassList0();  
    isDumpingArchive = isDumpingArchive0();  
    isSharingEnabled = isSharingEnabled0();  
}
```
其中涉及的三个方法都是native方法。