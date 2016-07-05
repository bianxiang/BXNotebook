## 8. OOP介绍

### 8.1 什么是OOP

#### 对象声明之前

C#中构造器加`new`关键字调用。

	CupOfCoffee myCup = new CupOfCoffee();
    CupOfCoffee myCup = new CupOfCoffee("Blue Mountain");

#### 静态和实例成员

可以定义一个静态构造器初始化静态成员。静态构造器只有一个。无参数。没有访问修饰符。静态构造器不能被直接调用。当下面情况之一发生时会被自动调用：

- 所在类的一个实例被创建
- 所在类的任一静态成员被访问

静态类只包含静态成员。不能被实例化。

### 8.2 OOP技术

#### 接口




