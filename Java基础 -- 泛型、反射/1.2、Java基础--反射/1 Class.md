

| 方法 | 返回值 |描述 |
| :----: |---- | ---- |
|getDeclaredClasses()|`Class<?>[]`|1)返回该类的public,protected,package,private权限的内部类,但不包含父类中的.</br>2)返回数组长度为0的情况:</br>- 该类本身没有继承类或实现接口.</br>- 该类是primitive类型或数组类型或void类型.|
# 获取接口

通过xxx.class.getInterfaces()方法获取当前Class实现或继承的接口。

需要注意的是：
1）只能获取到直接实现或继承的接口，而无法获取到层级关系之间的接口。

```java
public class Main {  
    public static void main(String[] args) {  
        System.out.println(ClassB.class.getInterfaces().length);  // 只能获取到InterfaceB接口
    }  
}  

interface InterfaceA{}  
interface InterfaceB{}  
  
  
class ClassA implements InterfaceA{}  
  
class ClassB extends ClassA implements InterfaceB{}
```


# 获取外部类

有两个方法能够获取类的外部类：
- getDeclaringClass：获取对应类的声明类Class对象。
- getEnclosingClass：获取对应类的直接外部类Class对象。

两者的区别在于匿名内部类的使用上、getEnclosingClass能够获取匿名内部类对应的外部类Class对象，而getDeclaringClass不能够获取匿名内部类对应的声明类Class对象。

```java
public class Main {  
    public static void main(String[] args) {  
        Class<?> enclosingClass = InnerClass.class.getEnclosingClass();  
        System.out.println(enclosingClass.getName());  // Main  
  
        Class<?> declaringClass = InnerClass.class.getDeclaringClass();  
        System.out.println(declaringClass.getName());  // Main  
  
        //注意：GetEnclosingClass获取的是直接定义该类的外部类Class实例、这点和getDeclaringClass一致  
        Class<?> enclosingClass1 = InnerClass.InnerInnerClass.class.getEnclosingClass();  
        System.out.println(enclosingClass1.getName());  // Main$InnerClass  
  
        Class<?> declaringClass1 = InnerClass.InnerInnerClass.class.getDeclaringClass();  
        System.out.println(declaringClass1.getName());  // Main$InnerClass  
  
        //针对匿名内部类的测试  
        DifferentBetweenClassGetEnclosingClassAndDeclaringClass s = new DifferentBetweenClassGetEnclosingClassAndDeclaringClass();  
        HelloService inst = s.getHelloService();  
        inst.sayHello();  
    }  
  
    private class InnerClass {  
        private class InnerInnerClass {  
        }  
    }  

    public interface HelloService {  
        void sayHello();  
    }  
    public static class DifferentBetweenClassGetEnclosingClassAndDeclaringClass {  
        HelloService getHelloService() {  
            //匿名内部类  
            return new HelloService() {  
                @Override
                public void sayHello() {  
	                // class Main_2$DifferentBetweenClassGetEnclosingClassAndDeclaringClass
                    System.out.println(this.getClass().getEnclosingClass());  
                    // null
                    System.out.println(this.getClass().getDeclaringClass());  
                }  
            };  
        }  
    }  
}
```

# 获取注解

有两个方法能获取标注在类上的注解：
- getAnnotations()
- getDeclaredAnnotations()

区别在于：前者能获取出现在类上的注解，其中包含从父类中继承的注解；后者只能获取直接标注在该类上的注解，也就是不能获取到从父类上继承的注解。

```java
public class Main {  
    public static void main(String[] args) {  
	    // @AnnotationA和@AnnotationB都能获取到
        System.out.println(B.class.getAnnotations().length);  
        // 只能获取到@AnnotationB
        System.out.println(B.class.getDeclaredAnnotations().length);  
    }  
}  
  
@Target({ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Inherited  
@interface AnnotationA{}  
@Target({ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@interface AnnotationB{}  
  
@AnnotationA  
class A{}  
  
@AnnotationB  
class B extends A{}
```




# Constructor部分

|        方法       |  描述|
| ----------------- | ------ |
| getConstructor(Class< ? > ...)|返回public指定构造方法，如果不存在或权限限定符不是public则抛出异常  |
| getDeclaredConstructor(Class< ? > ...) | 返回指定构造方法，如果不存在则抛出异常 |
| getConstructor() | 返回所有public构造方法 |
| getDeclaredContructor() | 返回所有构造方法 |

<font color=00FF00>expriments</font>

- 准备工作
```java
// package clazz;

public class Class1 {  
    public Class1(){}  
    Class1(int a){}  
    protected Class1(float b){}  
    private Class1(double c){}  
}
```

- 实验部分
```java
Class1.class.getConstructor(int.class)  // java.lang.NoSuchMethodException

Class1.class.getDeclaredConstructor(int.class) // protected clazz.Class1(float)

for(Constructor<?> con : Class1.class.getConstructors()){  
	System.out.println(con);  
}  
// public clazz.Class1()

for(Constructor<?> con : Class1.class.getDeclaredConstructors()){  
	System.out.println(con);  
}
// private clazz.Class1(double)
// protected clazz.Class1(float)
// clazz.Class1(int)
// public clazz.Class1()
```



# Field部分

| 方法 | 描述 |
| ----- | ----- |
| getField(String) | 返回给定string的public的属性 |
| getDeclaredField(String) | 返回给定string的属性 |
| getFields() | 返回public属性(包含继承而来的属性) |
| getDeclaredFields() | 返回所有属性(不包含继承而来的属性)

<font color=#00ff00>experiments</font>

- 准备工作
```java
// package clazz;

public class FieldClass {  
    public int filed0;  
    int filed1;  
    protected int field2;  
    private int field3;  
}
```

- 实验部分
```java


```


# Method部分

