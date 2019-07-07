---
layout: post
title: "【设计模式】里氏替换原则"
subtitle: 'Design Patterns - Liskov Substitution Principle'
author: "Chi"
date: 2019-07-04 22:23
header-style: text
catalog: false
tags:
  - Design Patterns
---

## 定义

> If for each object o1 of types S there is an object o2 of type of T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substitution for o2 then S is a subtype of T.

如果对每一个类型为S的对象o1，都有类型为T的对象o2，使得以T定义的所有程序P在所有的对象o1都替换成o2时，程序P的行为没有发生变化，那么类型S时类型T的子类型。

> Functoins that use pointers or references to base classes must be able to use objects of derived classes without knowing it.

所有引用基类的地方必须能够直接将引用的基类替换成其子类的对象，而不用去考虑子类的情况。

## 实现

- 子类必须完全实现父类的方法

  在类中调用其他类时务必要使用父类或接口，如果不能使用父类或接口，则说明类的设计违背了LSP原则。
  
  在具体的应用场景中要考虑子类是否能够完整地实现父类地业务。
  
  如果子类不能完整地实现父类的方法，或者父类的某些方法在子类中已经发生“畸变”，则建议断开父子继承关系，采用依赖，聚集，组合等关系代替继承。

  LSP 是继承复用的基石，只有当派生类可以替换掉基类，且软件单位的功能不受到影响时，基类才能真正被复用，而派生类也能够在基类的基础上增加新的行为。

- 子类可以有自己的个性

  在上文中我们总结了，父类出现的地方，可以将父类直接替换成他的子类，那么反过来就不成立了，因为子类有它自己的方法和属性。

- 重载或实现父类的方法时输入参数可以被放大

  在重载或实现父类的方法时，方法的参数范围可以放大，这时用子类代替父类传递到调用者中，由于子类的参数范围比父类大，所以子类的方法永远不会被执行，执行的是输入参数范围小的父类的方法。这样做的目的就是，避免在将父类替换成其子类的时候，本该调用父类中的方法而调用了子类中重载的方法，引发业务逻辑混乱。

  总的来说，这种做法就是** 所有引用基类的地方必须能够直接将引用的基类替换成其子类的对象，而不用去考虑子类的情况**的具体实现。

## 总结

里氏代换原则是对开闭原则的补充。实现开闭原则的关键步骤就是抽象化，而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

采用里氏替换原则的目的就是增强程序的健壮性，版本升级时也可以保持非常好的兼容性。

在项目中，采用里氏替换原则时，尽量避免子类的“个性”。

## 参考资料

秦小波. 设计模式之禅（第2版）[M]. 机械工业出版社, 2014.

[设计模式简介](https://www.runoob.com/design-pattern/design-pattern-intro.html)
