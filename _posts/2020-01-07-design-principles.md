---
layout: post
title: "【设计模式】设计原则--面向接口编程你理解的对吗？"
subtitle: 'Design Patterns - Design Principles'
author: "Chi"
date: 2020-01-07 14:00
header-style: text
catalog: false
tags:
  - Design Patterns
---

最近看了《Head First Design Patterns》这本书。正如其名，这本书讲的是设计模式（Design Patterns），而这本书的第一章，讲的是很重要的一些设计原则（Design Principles）。

- Identify the aspects of your application that vary and separate them from what stays the same.（识别应用程序中各个方面的变化，并将它们与保持不变的部分分开。）

- Program to an interface, not an implementation.（面向接口而不是实现编程。）

- Favor composition over inheritance.（优先考虑组成而不是继承。）

其中令我感触颇深的是，“面向接口而不是实现编程”颠覆了我一直以来的认识。

## 开始

书中用了一个很形象的示例：模拟鸭子程序（SimUDuck）。系统的最初设计使用标准的OO技术，并创建了一个Duck基类，所有其他Duck类型都继承自该基类。

![Duck Class Diagram](/img/in-post/2020-01-07-design-principles/duck-class-diagram.jpg)

设计系统时考虑到鸭子都会发出叫声，而且都会游泳，于是将`Quack`方法和`Swim`方法定义到`Duck`基类中并实现；此外，并不是所有的鸭子都是长得一样的，那么将`Display`方法在`Duck`基类中定义为抽象的，所有继承自`Duck`基类的子类编写自己的实现。

新的需求产生了！我们需要让系统中的鸭子可以飞。从面向对象的角度来考虑，如果我们想要**代码重用**，只需要在`Duck`基类中添加方法`Fly`并实现它——所有的鸭子子类都是继承自`Duck`基类的——就实现了让鸭子飞的功能。我们通过继承实现了**代码重用**，很轻松就解决了问题。

也许我们需要深入考虑一下，所有的鸭子都会飞吗？玩具橡胶鸭呢？我们把`Fly`方法的定义及实现放到了`Duck`基类中，所有继承自它的子类都继承到了`Fly`方法，其中也包括了不应继承`Fly`方法的子类。如果按照上面的方案，那我们只能在`RubberDuck`橡胶鸭子类中重写父类的`Fly`方法让`RubberDuck`执行`Fly`的时候什么都不做。

再深入一些，如果我们的系统中除了橡胶鸭外，还有其他各种鸭子，比如木头鸭子呢？这时`DecoyDuck`木头鸭子继承来的`Quack`方法出现了问题——木头鸭子不会叫！我们只好再把`DecoyDuck`中的`Quack`方法重写了......

如果我们改用接口会怎么样呢？把`Quack`和`Fly`方法从基类中拿出来，分别在`IQuackable`和`IFlyable`接口中定义，然后我们不同的子类根据需要来继承接口，并实现`Quack`或`Fly`方法。

![Interface Class Diagram](/img/in-post/2020-01-07-design-principles/duck-interface.jpg)

当我们有很多个子类鸭子的时候，就要分别为每个继承了`IQuackable`或`IFlyable`接口的子类来编写`Quack`或`Fly`的实现方法，这完全破坏了**代码重用**!值得注意的是，虽然我们在这里使用了接口，但这并不是面向接口编程。

## 封装变化

这里引入第一条设计原则：***Identify the aspects of your application that vary and separate them from what stays the same.（识别应用程序中各个方面的变化，并将它们与保持不变的部分分开。）***

换言之：*take the parts that vary and encapsulate them, so that later you can alter or extend the parts that vary without affecting those that don’t.（将变化的部分封装起来，以便以后可以更改或扩展变化的部分而不会影响那些不变的部分。）*

这样带来的好处是，我们可以进行更少的代码更改来实现需求功能，减少因代码更改而带来的意想不到的影响，并且提高了系统灵活性。

我们知道`Duck`的不同子类中，`Quack`和`Fly`的行为是会发生变化的，那么我们将`Quack`和`Fly`方法从`Duck`基类中拿出来，并为`Quack`和`Fly`方法分别创建一些类，来实现各种不同的行为。

![Encapsulation Varies](/img/in-post/2020-01-07-design-principles/encapsulation-varies.jpg)

## 面向接口而不是实现编程

设计原则：***Program to an interface, not an implementation.（面向接口而不是实现编程。）***

在这里，面向接口而不是实现编程，和封装变化是相辅相成的。值得注意的是，这里所说的接口，

我们可以定义接口来代表行为：`IFlyBehavior`,`IQuackBehavior`。接口的实现则是行为具体的表现形式。我们可以将接口的不同实现类，来赋值给`Duck`的不同子类。

coming soon...
