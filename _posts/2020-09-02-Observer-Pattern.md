---
layout: post
title: "Observer Pattern"
subtitle: "Keeping your Objects in the know"
author: "Chi"
date: 2020-09-02 15:00
header-style: text
catalog: true
tags:
  - Design Patterns
---

## The Observer Pattern defined

The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are
notified and updated automatically.

> Publishers + Subcribers = Observer Pattern

With the Observer pattern, the Subject is the object that contains the state and controls it. So, there is **ONE** subject with state. The observers, on the other hand, use the state, even if they don’t own it. There are **MANY** observers and they rely on the Subject to tell them when its state changes. So there is a relationship between the **ONE** Subject to the **MANY** Observers.

## The power of Loose Coupling

When two objects are loosely coupled, they can interact, but have very little knowledge of each other.

The Observer Pattern provides an object design where subjects and observers are loosely coupled. Because:

- The only thing the subject knows about an observer is that it implements a certain interface.
- We can add new observers at any time.
- We never need to modify the subject to add new types of observers.
- We can reuse subjects or observers independently of each other.
- Changes to either the subject or an observer will not affect the other.

Loosely coupled designs allow us to build flexible OO systems that can handle change because they minimize the interdependency between objects.

## Design Pricple

Strive for loosely coupled designs between objects that interact.

## Using of Design Pricple

### Encapsulated vary

> Identify the aspects of your application that vary and separate them from what stays the same.

The thing that varies in the Observer Pattern is the state of the Subject and the number and types of Observers. With this pattern, you can vary the objects that are dependent on the state of the Subject, without having to change that Subject. That’s called planning ahead!

### Program to an interface, not an implementation

Both the Subject and Observer use interfaces. The Subject keeps track of objects implementing the Observer interface, while the observers register with, and get notified by, the Subject interface. As we’ve seen, this keeps things nice and loosely coupled.

### Favor composition over inheritance

The Observer Pattern uses composition to compose any number of Observers with their Subjects. These relationships aren’t set up by some kind of inheritance hierarchy. No, they are set up at runtime by composition!

## Reference

> Freeman E, Robson E, Bates B, et al. Head first design patterns[M]. " O'Reilly Media, Inc.", 2008.
