
# 获取注解的属性

借用Spring获取注解属性的方法。
```java
Method[] methods = annotationType.getDeclaredMethods();  
int size = methods.length;  
for (int i = 0; i < methods.length; i++) {  
   if (!isAttributeMethod(methods[i])) {  
      methods[i] = null;  
      size--;  
   }  
}  
if (size == 0) {  
   return NONE;  
}  
Arrays.sort(methods, methodComparator);  
Method[] attributeMethods = Arrays.copyOf(methods, size);  
return new AttributeMethods(annotationType, attributeMethods);
```