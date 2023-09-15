# 一、invokespecial

invokespecial通常根据引用的类型选择方法，而不是对象的类来选择！即它使用静态绑定而不是动态绑定。

使用invokespecial指令分为下面三种情况：

- 实例的初始化方法（"< init >"）。
- 私有方法。
- super关键字调用的方法。

因为上述三种方法能在编译期间就确定。