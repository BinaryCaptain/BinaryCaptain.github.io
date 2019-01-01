---
layout: post
keywords: blog
description: blog
title: "设计模式入门"
tags: [设计]
date: 2017-01-11
categories: 设计模式入门
cover: 'https://binarycaptain.github.io/assets/img/design.jpg'
tags: 设计模式入门
subtitle: '设计模式入门'

---


## 设计模式入门

1. 设计模式概念

设计模式不是代码，而是解决问题的方案，学习现有的设计模式可以做到经验复用。

拥有设计模式词汇，在沟通时就能用更少的词汇来讨论，并且不需要了解底层细节。

2. 问题描述

设计不同种类的鸭子拥有不同的叫声和飞行方式。

3. 简单实现方案

使用继承的解决方案如下，这种方案代码无法复用，如果两个鸭子类拥有同样的飞行方式，就有两份重复的代码。

![](https://binarycaptain.github.io/assets/img/design.jpg)

4. 设计原则

封装变化在这里变化的是鸭子叫和飞行的行为方式。

针对接口编程，而不是针对实现编程 变量声明的类型为父类，而不是具体的某个子类。父类中的方法实现不在父类，而是在各个子类。程序在运行时可以动态改变变量所指向的子类类型。

运用这一原则，将叫和飞行的行为抽象出来，实现多种不同的叫和飞行的子类，让子类去实现具体的叫和飞行方式。

![](https://binarycaptain.github.io/assets/img/design-1.png)

多用组合，少用继承 组合也就是 has-a 关系，通过组合，可以在运行时动态改变实现，只要通过改变父类对象具体指向哪个子类即可。而继承就不能做到这些，继承体系在创建类时就已经确定。

运用这一原则，在 Duck 类中组合 FlyBehavior 和 QuackBehavior 类，performQuack() 和 performFly() 方法委托给这两个类去处理。通过这种方式，一个 Duck 子类可以根据需要去实例化 FlyBehavior 和 QuackBehavior 的子类对象，并且也可以动态地进行改变。

![](https://binarycaptain.github.io/assets/img/design-2.jpg)

5. 整体设计图

![](https://binarycaptain.github.io/assets/img/design-3.jpg)

6. 模式定义

策略模式 ：定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

7. 实现代码

```java
public abstract class Duck {
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;

    public Duck(){
    }

    public void performFly(){
        flyBehavior.fly();
    }

    public void setFlyBehavior(FlyBehavior fb){
        flyBehavior = fb;
    }

    public void performQuack(){
        quackBehavior.quack();
    }

    public void setQuackBehavior(QuackBehavior qb){
        quackBehavior = qb;
    }
}
```
```java
public class MallarDuck extends Duck{
    public MallarDuck(){
        flyBehavior = new FlyWithWings();
        quackBehavior = new Quack();
    }
}
```
```java
public interface FlyBehavior {
    void fly();
}
```
```java
public class FlyNoWay implements FlyBehavior{
    @Override
    public void fly() {
        System.out.println("FlyBehavior.FlyNoWay");
    }
}
```
```java
public class FlyWithWings implements FlyBehavior{
    @Override
    public void fly() {
        System.out.println("FlyBehavior.FlyWithWings");
    }
}
```
```java
public interface QuackBehavior {
    void quack();
}
```
```java
public class Quack implements QuackBehavior{
    @Override
    public void quack() {
        System.out.println("QuackBehavior.Quack");
    }
}
```
```java
public class MuteQuack implements QuackBehavior{
    @Override
    public void quack() {
        System.out.println("QuackBehavior.MuteQuack");
    }
}
```

```java
public class Squeak implements QuackBehavior{
    @Override
    public void quack() {
        System.out.println("QuackBehavior.Squeak");
    }
}

```java
public class MiniDuckSimulator {
    public static void main(String[] args) {
        Duck mallarDuck = new MallarDuck();
        mallarDuck.performQuack();
        mallarDuck.performFly();
        mallarDuck.setFlyBehavior(new FlyNoWay());
        mallarDuck.performFly();
    }
}
```
执行结果

```java
QuackBehavior.Quack
FlyBehavior.FlyWithWings
FlyBehavior.FlyNoWay
```

## 第二章 观察者模式

1. 模式定义

定义了对象之间的一对多依赖，当一个对象改变状态时，它的所有依赖者都会受到通知并自动更新。主题（Subject）是被观察的对象，而其所有依赖者（Observer）成为观察者，前端MVC框架当中model层数据更新了之后，使用观察者模式去通知view层。

![](https://binarycaptain.github.io/assets/img/design-4.jpg)

2. 模式类图

主题中具有注册和移除观察者，并通知所有注册者的功能，主题是通过维护一张观察者列表来实现这些操作的。

观察者拥有一个主题对象的引用，因为注册、移除还有数据都在主题当中，必须通过操作主题才能完成相应功能。

![](https://binarycaptain.github.io/assets/img/design-5.jpg)

3. 问题描述

天气数据布告板会在天气信息发生改变时更新其内容，布告板有多个，并且在将来会继续增加。

4. 解决方案类图

![](https://binarycaptain.github.io/assets/img/design-6.jpg)

5. 设计原则

为交互对象之间的松耦合设计而努力 当两个对象之间松耦合，它们依然可以交互，但是不太清楚彼此的细节。由于松耦合的两个对象之间互相依赖程度很低，因此系统具有弹性，能够应对变化。

6. 实现代码

```java
public interface Subject {
    public void resisterObserver(Observer o);
    public void removeObserver(Observer o);
    public void notifyObserver();
}
```
```java
import java.util.ArrayList;
import java.util.List;

public class WeatherData implements Subject {
    private List<Observer> observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList<>();
    }

    @Override
    public void resisterObserver(Observer o) {
        observers.add(o);
    }

    @Override
    public void removeObserver(Observer o) {
        int i = observers.indexOf(o);
        if (i >= 0) {
            observers.remove(i);
        }
    }

    @Override
    public void notifyObserver() {
        for (Observer o : observers) {
            o.update(temperature, humidity, pressure);
        }
    }

    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        notifyObserver();
    }
}
```
```java
public interface Observer {
    public void update(float temp, float humidity, float pressure);
}
```
```java
public class CurrentConditionsDisplay implements Observer {
    private Subject weatherData;

    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.resisterObserver(this);
    }

    @Override
    public void update(float temp, float humidity, float pressure) {
        System.out.println("CurrentConditionsDisplay.update:" + temp + " " + humidity + " " + pressure);
    }
}
```
```java
public class StatisticsDisplay implements Observer {
    private Subject weatherData;

    public StatisticsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.resisterObserver(this);
    }

    @Override
    public void update(float temp, float humidity, float pressure) {
        System.out.println("StatisticsDisplay.update:" + temp + " " + humidity + " " + pressure);
    }
}
```
```java
public class WeatherStation {
    public static void main(String[] args) {
        WeatherData weatherData = new WeatherData();
        CurrentConditionsDisplay currentConditionsDisplay = new CurrentConditionsDisplay(weatherData);
        StatisticsDisplay statisticsDisplay = new StatisticsDisplay(weatherData);

        weatherData.setMeasurements(0, 0, 0);
        weatherData.setMeasurements(1, 1, 1);
    }
}
```
执行结果
```java
CurrentConditionsDisplay.update:0.0 0.0 0.0
StatisticsDisplay.update:0.0 0.0 0.0
CurrentConditionsDisplay.update:1.0 1.0 1.0
StatisticsDisplay.update:1.0 1.0 1.0.0
```
## 第三章 装饰模式

1. 问题描述

设计不同种类的饮料，并且每种饮料可以动态添加新的材料，比如可以添加牛奶。计算一种饮料的价格。

2. 模式定义

动态地将责任附加到对象上。在扩展功能上，装饰者提供了比继承更有弹性的替代方案。

下图中 DarkRoast 对象被 Mocha 包裹，Mocha 对象又被 Whip 包裹，并且他们都继承自相同父类，都有 cost() 方法，但是外层对象的 cost() 方法实现调用了内层对象的 cost() 方法。因此，如果要在 DarkRoast 上添加 Mocha，那么只需要用 Mocha 包裹 DarkRoast，如果还需要 Whip ，就用 Whip 包裹 Mocha，最后调用 cost() 方法能把三种对象的价格都包含进去。

![](https://binarycaptain.github.io/assets/img/design-7.jpg)

3. 模式类图

装饰者和具体组件都继承自组件类型，其中具体组件的方法实现不需要依赖于其它对象，而装饰者拥有一个组件类型对象，这样它可以装饰其它装饰者或者具体组件。所谓装饰，就是把这个装饰者套在被装饰的对象之外，从而动态扩展被装饰者的功能。装饰者的方法有一部分是自己的，这属于它的功能，然后调用被装饰者的方法实现，从而也保留了被装饰者的功能。可以看到，具体组件应当是装饰层次的最低层，因为只有具体组件有直接实现而不需要委托给其它对象去处理。

![](https://binarycaptain.github.io/assets/img/design-8.jpg)

4. 问题解决方案的类图
![](https://binarycaptain.github.io/assets/img/design-9.jpg)

5. 设计原则

类应该对扩展开放，对修改关闭。 也就是添加新功能时不需要修改代码。在本章问题中该原则体现在，在饮料中添加新的材料，而不需要去修改饮料的代码。观察则模式也符合这个原则。不可能所有类都能实现这个原则，应当把该原则应用于设计中最有可能改变的地方。

6. Java I/O 中的装饰者模式

![](https://binarycaptain.github.io/assets/img/design-10.jpg)

































