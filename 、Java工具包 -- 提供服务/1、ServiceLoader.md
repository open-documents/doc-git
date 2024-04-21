
>参考：JDK 18.0.1 ServiceLoader源码

<span style="color:#00E0FF">ServiceLoader</span>用来动态加载接口（称为<span style="color:#FF5500">服务</span>）的实现类（称为<span style="color:#FF5500">服务提供者</span>）。

简单理解，<span style="color:#FF5500">服务</span>是接口或者是类，<span style="color:#FF5500">服务提供者</span>是接口实现类或者是类子类。一个<span style="color:#FF5500">服务</span>可以有多个服务提供者，应用程序的引用对象是<span style="color:#FF5500">服务</span>，应用通过<span style="color:#00E0FF">ServiceLoader</span>运行时获取<span style="color:#FF5500">服务提供者</span>。

<span style="color:#00E0FF">ServiceLoader</span>提供<span style="color:#44cf57">load(Class)</span>及重载方法来获取<span style="color:#00E0FF">ServiceLoader</span>实例。

# 初识 ServiceLoader

下面通过一个例子来理解<span style="color:#00E0FF">ServiceLoader</span>的作用，准备下面的环境。。

工程module_test，源码目录main，<span style="color:#FF5500">服务</span>（Service）、<span style="color:#FF5500">服务提供者</span>（ServiceProviderOne、ServiceProviderTwo）
```text
module_test
    |----main
           |----services
           |        |----Service.java
           |        |----ServiceProviderOne.java
           |        |----ServiceProviderTwo.java
           |----Main.java
```
```java
package main.services;  
public interface Service {  
    void String getName();  
}
public class ServiceProviderOne implements Service {  
    @Override  
    public void getName() {  return "main.services.ServiceProviderOne";  }  
}
public class ServiceProviderTwo implements Service {  
    @Override  
    public void getName() {  return "main.services.ServiceProviderTwo";  }  
}
```

---
）<span style="color:#00E0FF">ServiceLoader</span>实例提供<span style="color:#44cf57">iterator()</span>来遍历所有的<span style="color:#FF5500">服务提供者</span>。
```java
public class Main {  
    public static void main(String[] args) {  
        ServiceLoader<Service> services = ServiceLoader.load(Service.class);  
        for(Service service : services){  
            System.out.println(service.getName()); 
        }  
    }  
}
```
输出：
```text
main.services.ServiceProviderOne
main.services.ServiceProviderTwo
```

）<span style="color:#00E0FF">ServiceLoader</span>实例提供<span style="color:#44cf57">stream()</span>来流式所有的<span style="color:#FF5500">服务提供者</span>，但是不会实例化<span style="color:#FF5500">服务提供者</span>。<span style="color:#44cf57">stream</span>中包含的元素类型是<span style="color:#00E0FF">ServiceLoader</span>.Provider，提供<span style="color:#44cf57">type()</span>获取<span style="color:#FF5500">服务提供者</span>的类型，提供<span style="color:#44cf57">get()</span>获取<span style="color:#FF5500">服务提供者</span>的实例。

```java
public class Main {  
    public static void main(String[] args) {  
		ServiceLoader<Service> services = ServiceLoader.load(Service.class);  
		Set<Service> serviceSet = services  
		        .stream()  
		        .filter(serviceProvider 
			         -> serviceProvider.get().printName().equals("main.services.ServiceProviderTwo"))  
		        .map(ServiceLoader.Provider::get)  
		        .collect(Collectors.toSet());
    }  
}
```

# 在模组中部署服务提供者

1）如果应用是<span style="color:#FFDD00">模组</span>，模组声明中<span style="color:#fb463c">必须</span>使用`uses <service-name>`指定需要使用的<span style="color:#FF5500">服务</span>，需要使用`provides <service-name> with <service-provider-name>`指定提供的<span style="color:#FF5500">服务</span>对应的<span style="color:#FF5500">服务提供者</span>。
2）如果<span style="color:#FFDD00">模组</span>中的包不包含该服务，需要使用`reuqires <module-name>`依赖其它包含该服务的<span style="color:#FFDD00">模组</span>。（容易理解，模组中不包含<span style="color:#FF5500">服务</span>接口，代码会直接报错）
3）强烈建议<span style="color:#FFDD00">模组</span><span style="color:#fb463c">不要依赖</span>包含<span style="color:#FF5500">服务提供者</span>的其它模组。（个人理解，如果模组自身包含<span style="color:#FF5500">服务</span>，那么如果其它模组想要包含<span style="color:#FF5500">服务提供者</span>则因需要导入<span style="color:#FF5500">服务</span>而依赖前者模组，而前者模组如果因需要导入非本模组中的<span style="color:#FF5500">服务提供者</span>而依赖后者模组会造成循环依赖）

使用下面情境帮助理解。

父模组（module_test），子模组1（ module_test1），子模组2（module_test2），子模组1包含<span style="color:#FF5500">服务</span>及<span style="color:#FF5500">服务提供者</span>。
```text
module_test
    |----main
    |       |----services
    |       |        |----Service.java
    |       |        |----ServiceProviderOne.java
    |       |        |----ServiceProviderTwo.java
    |----module-info.java
```

---
测试1：在<span style="color:#FFDD00">模组</span>中使用<span style="color:#FF5500">服务</span>时，在模组声明中不通过`uses <service-name>`指定<span style="color:#FF5500">服务</span>。

）子模组1声明如下。
```java
module test.module.one{}  
```
）子模组1的Main中使用ServiceLoader动态加载服务。
```java
public class Main {  
    public static void main(String[] args) {  
        ServiceLoader<Service> services = ServiceLoader.load(Service.class);  
        for(Service service : services){  
            System.out.println(service.getName()); 
        }  
    }  
}
```
）出现如下异常。
```text
Exception in thread "main" java.util.ServiceConfigurationError: main_one.services.Service: module test.module.one does not declare `uses`
```

---
测试2：在<span style="color:#FFDD00">模组</span>中使用<span style="color:#FF5500">服务</span>时，在模组声明中只指定<span style="color:#FF5500">服务</span>，不指定<span style="color:#FF5500">服务提供者</span>。

）修改测试1中子模组1声明。
```java
module test.module.one{  
	uses main_one.services.Service;
}
```
）Main没有输出。

---
测试3：在<span style="color:#FFDD00">模组</span>中使用<span style="color:#FF5500">服务</span>时，在模组声明中既指定<span style="color:#FF5500">服务</span>，也指定<span style="color:#FF5500">服务提供者</span>。

）修改测试1中子模组1声明。
```java
module test.module.one{  
	uses main_one.services.Service;
	provides main_one.services.Service with 
	    main_one.services.ServiceProviderOne, 
	    main_one.services.ServiceProviderTwo;  
}
```
）Main输出如下。
```text
test.services.ServiceProviderOne
test.services.ServiceProviderTwo
```

# 在类路径中部署服务提供者

获取jar包中的<span style="color:#FF5500">服务</span>对应的<span style="color:#00E0FF">ServiceLoader</span>，需要1）在jar包中的`META-INF/services`目录下新建名称<span style="color:#FF5500">服务</span>全限定名的文件；2）文件中的每行对应一个<span style="color:#FF5500">服务提供者</span>的全限定名。

以