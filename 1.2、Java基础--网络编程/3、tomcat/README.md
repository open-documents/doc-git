**
# 一、Tomcat与Java

1、Tomcat与Java SE

Tomcat是用Java语言编写的，需要运行在Java虚拟机上，所以一般需要先安装JDK，以提供运行环境。

2、Tomcat与Java EE

Tomcat实现了几个Java EE规范，包括 `Java Servlet`、`Java Server Pages（JSP）`，`Java Expression Language` 和 `Java WebSocket` 等，这些是都下载Tomcat安装包默认提供的，可以在源码中看到相关Java EE 规范API源码引用。

# 二、Tomcat与Servlet/编程开发

Tomcat实现的几个Java EE规范最重的是Servlet，因为实现了Servlet规范，所以Tomcat也是一个Servlet容器，可以运行我们自己编写的Servlet应用程序处理动态请求。

平时用的Struts2、SpringMVC框架就是基于Servlet，所以我们可以在这些框架的基础上进行快速开发，然后部署到Tomcat中运行。

另外对于JSP规范实现：可以混合HTML与Java开发在一个文件中（.jsp），这种文件在第一次调用之后会被Tomcat的Jasper组件编译成一个servlet类，在后续的操作中则可以直接使用此类。这种开发方式不灵活，一般少用。





参考：https://blog.csdn.net/tjiyu/article/details/54590258
