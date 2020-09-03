---
layout: post
title: "Factory Pattern"
subtitle: "Baking with OO Goodness"
author: "Chi"
date: 2020-09-03 12:00
header-style: text
catalog: true
tags:
  - Design Patterns
---

## Identifying the aspects that vary

When we need to determine which type to instantiate based on certain conditions, dealing with which concrete class is instantiated is really messing up our  method and preventing it from **being closed for modification**.

Identifying the aspects that vary during the instantiation of the object and encapsulate it.

Factories handle the details of object creation.

## Factory Method Pattern

All factory patterns encapsulate object creation. The Factory Method Pattern encapsulates object creation by letting subclasses decide what objects to create.

The factory method is the key to encapsulating product knowledge, and decoupling the implementation of the product from its use.

### Factory Method Pattern defined

**The Factory Method Pattern defi nes an interface for creating an object, but lets subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.**

As with every factory, the Factory Method Pattern gives us a way to encapsulate the instantiations of concrete types.

As in the official definition, you’ll often hear developers say that the Factory Method lets subclasses decide which class to instantiate. They say “decides” not because the pattern allows subclasses themselves to decide **at runtime**, but because the creator class is written without knowledge of the actual products that will be created, which is decided purely by the **choice** of the subclass that is used.

![Factory Method Pattern](/img/in-post/2020-09-03-Factory-Pattern/factory-method-pattern.png)

### Compare with Simple Factory

The Factory Method Pattern has defined a `create` method as an abstract method. It is up to each subclass to define the behavior of the `create` method. In Simple Factory, the factory is another object that is **composed** with the `create` method.

With Factory Method you are creating a framework that let’s the subclasses decide which implementation will be used. Compare that with SimpleFactory, which gives you a way to encapsulate object creation, but doesn’t give you the flexibility of the Factory Method because there is no way to vary the products you’re creating. Think about Open-Close Principle.

## The Dependency Inversion Principle

**Depend upon abstractions. Do not depend upon concrete classes.**

It suggests that our high-level components should not depend on our low-level components; rather, they should both depend on abstractions.

- No variable should hold a reference to a concrete class.
- No class should derive from a concrete class.
- No method should override an implemented method of any of its base classes.

## Abstract Factory Pattern defined

**The Abstract Factory Pattern provides an interface for creating families of related or dependent objects without specifying their concrete classes.**

## Factory Method and Abstract Factory compared

- Factory Method

![Compare - Factory Method](/img/in-post/2020-09-03-Factory-Pattern/compare-factory-method.png)

- Abstract Factory

![Compare - Factory Method](/img/in-post/2020-09-03-Factory-Pattern/compare-abstract-factory.png)

## Reference

> Freeman E, Robson E, Bates B, et al. Head first design patterns[M]. " O'Reilly Media, Inc.", 2008.
