public interface Shape { void draw(); }

### 步骤 2

创建实现接口的实体类。

## Rectangle.java

public class Rectangle implements Shape { @Override public void draw() { System.out.println("Shape: Rectangle"); } }

## Circle.java

public class Circle implements Shape { @Override public void draw() { System.out.println("Shape: Circle"); } }

### 步骤 3

创建实现了 _Shape_ 接口的抽象装饰类。

## ShapeDecorator.java

public abstract class ShapeDecorator implements Shape { protected Shape decoratedShape; public ShapeDecorator(Shape decoratedShape){ this.decoratedShape = decoratedShape; } public void draw(){ decoratedShape.draw(); } }

### 步骤 4

创建扩展了 _ShapeDecorator_ 类的实体装饰类。

## RedShapeDecorator.java

public class RedShapeDecorator extends ShapeDecorator { public RedShapeDecorator(Shape decoratedShape) { super(decoratedShape); } @Override public void draw() { decoratedShape.draw(); setRedBorder(decoratedShape); } private void setRedBorder(Shape decoratedShape){ System.out.println("Border Color: Red"); } }

### 步骤 5

使用 _RedShapeDecorator_ 来装饰 _Shape_ 对象。

## DecoratorPatternDemo.java

public class DecoratorPatternDemo { public static void main(String[] args) { Shape circle = new Circle(); ShapeDecorator redCircle = new RedShapeDecorator(new Circle()); ShapeDecorator redRectangle = new RedShapeDecorator(new Rectangle()); //Shape redCircle = new RedShapeDecorator(new Circle()); //Shape redRectangle = new RedShapeDecorator(new Rectangle()); System.out.println("Circle with normal border"); circle.draw(); System.out.println("\nCircle of red border"); redCircle.draw(); System.out.println("\nRectangle of red border"); redRectangle.draw(); } }

### 步骤 6

执行程序，输出结果：

Circle with normal border
Shape: Circle

Circle of red border
Shape: Circle
Border Color: Red

Rectangle of red border
Shape: Rectangle
Border Color: Red