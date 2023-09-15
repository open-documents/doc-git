PropertyEditor在Spring中被用来将String类型转换为合适的对象实例。

PropertyEditor有一个实现类 java.beans.PropertyEditorSupport 。

在PropertyEditorSupport中，维护一个PropertyChangeListerner的Vector，用来存储所有的属性改变监听器，当属性发生改变时，会发布事件给该vector存储的所有监听器。

# PropertyEditorSupport

需要自定义PropertyEditor，通常直接继承PropertyEditorSupport即可。

PropertyEditorSupport内部有3个重要属性。
```java
private Object value;  
private Object source;  
private java.util.Vector<PropertyChangeListener> listeners;
```
value：被改变后的属性对象。
source：用于属性改变后事件中的源。
listerners：存储所有的属性改变监听器。
-- -- 
## 1）需要重写的方法

*setAsString(String)*

一般需要通过重写setAsString()方法，在该方法中对String进行解析并生成期望的对象，在内部调用setValue()方法，将value设置成生成的期望对象。

## 2）需要了解的方法

*setValue(Object)*：内部除了设置value值，还会触发firePropertyChange()方法。

*firePropertyChange()*：内部向listerners的每个监听器发布事件。

*addPropertyChangeListener(PropertyChangeListener)*：向listerners添加监听器。

*removePropertyChangeListener(PropertyChangeListener)*：从listerners中删除指定的监听器。






