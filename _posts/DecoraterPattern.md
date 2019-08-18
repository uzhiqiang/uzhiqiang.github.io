# 装饰者模式

## 概述

采用装饰者模式，可以在不修改任何底层代码的情况下，给原有的对象赋予新的功能。



## 设计原则

**类应该对扩展开放，对修改关闭。**

该原则也称为`开放-关闭`原则，它可以使得类易于扩展，在不修改代码的情况下，原有的类可以搭配新的行为。遵循`开放-关闭`原则，通常会引入新的抽象层次，增加代码复杂度。因此，需要把注意力集中在设计中最有可能改变的地方，然后应用开放-关闭原则，没有必要每个地方都采用该原则。



## 模式定义

动态地将责任附加到对象上。若要扩展功能，装饰者可提供了比继承更有弹性的替代方案。

![decorator](/Users/uzhiqiang/设计模式笔记/out/figure/decorator/decorator.svg)

- `ConcreteComponent`

  我们将要在该对象上动态地加上新行为，它继承自`Component`。

- `Decorator`

  它是装饰者的基类，它要继承自`Component`，并且还要包含一个`Component`成员，这是它能实现装饰的重要原因。

  `Decorator`所包含的一个`Component`成员是用来记录一层嵌套一层的装饰信息。所谓装饰，其实就是将装饰对象赋值给`ConcreteDecorator`的`WrappedObj`。在所有`ConcreteDecorator`的`methodA()`和`methodB()`中，都要委托`WrappedObj`调用它的`methodA()`和`methodB()`，并且通常还会在其前或后加上自己东西。利用多态特性，最外层装饰者调用`methodA()`会去调用次外层装饰者的`methodA()`，如此不断，直到最内层的`ConcreteComponent`的`methodA()`。

  `ConcreteComponent`是被装饰的对象，装饰者本身也可以是被装饰的对象。例如用`ConcreteDecoratorA`装饰`ConcreteComponent`，再用`ConcreteDecoratorB`装饰`ConcreteDecoratorA`。也就是说，装饰者“反映”了它所装饰的对象，这里的“反映”指的是装饰者和被装饰者类型一致，因此`Decorator`要继承自`Component`。

- `ConcreteDecorator`

  具体的装饰者可以加上新的方法



## Demo

设计一个咖啡店的订单系统，该店的原料（咖啡和调料）如下表所示：

|        咖啡类型        |  规格  | 价格 |
| :--------------------: | :----: | :--: |
| HouseBlend（综合咖啡） | 标准杯 | 0.89 |
| Decaf（低咖啡因咖啡）  | 标准杯 | 1.05 |
| DarkRoast（深焙咖啡）  | 标准杯 | 0.99 |
|  Espresso（浓缩咖啡）  | 标准杯 | 1.99 |


|   调料类型    | 价格 |
| :-----------: | ---- |
| Milk（牛奶）  | 0.1  |
| Mocha（摩卡） | 0.2  |
|  Soy（豆浆）  | 0.15 |
| Whip（奶泡）  | 0.1  |



### 需求

咖啡店的订单系统具体需求如下：

- 调料可以随意添加
- 小票需要打印中顾客订的咖啡的所有原料

### 需求分析及设计思路

根据装饰者模式的原理，以饮料（咖啡）为主体，然后在运行时以调料来装饰饮料，各个类的设计如下图所示。

![CoffeOrderSystem](/Users/uzhiqiang/设计模式笔记/out/figure/CoffeOrderSystem/CoffeOrderSystem.svg)

需要注意的是，`CondimentDecorator`里的`GetDescription`覆盖了`Beverage`的。`Milk`、`Mocha`、`Soy`和`Whip`的`GetDescription()`方法和`cost()`方法都要委托`beverage`指针调用被它装饰的对象的的`GetDescription()`方法和`cost()`方法，并且还要在其前或后加上自己东西。

`Beverage`中的`GetDescription`如下：

```c++
std::string Beverage::GetDescription() {
    return description;
}
```

具体调料的`GetDescription`如下，以`Milk`为例：

```c++
std::string Milk::GetDescription() {
    return beverage->GetDescription() + " Milk";
}
```



比方说，若顾客想要一杯摩卡和奶泡的低咖啡因咖啡，那么运行的步骤如下：

1. 创建一个`Decaf`对象
2. 以`Mocha`对象来装饰它
3. 以`Whip`对象来装饰它
4. 调用`cost()`方法，并依赖委托将调料价钱加上去

```c++
int main() {
  Decaf decaf;
  Beverage *beverage = &decaf;
  Mocha ma(beverage);  //用Mocha来装饰Decaf
  beverage = &ma;
  Whip whip(beverage); //用Whip来装饰Mocha
  beverage = &whip;
  std::cout << beverage->GetDescription() + ", cost: " << beverage->cost() << std::endl;
  return 0;
}
```

输出

```shell
Decaf Mocha Whip, cost: 1.35
```



### 代码