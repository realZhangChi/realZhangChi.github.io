---
layout: post
title: "Decorator Pattern"
subtitle: "Decorating Objects"
author: "Chi"
date: 2020-09-03 10:00
header-style: text
catalog: true
tags:
  - Design Patterns
---

Once you know the techniques of decorating, you’ll be able to give your (or someone else’s) objects new responsibilities without making any code changes to the underlying classes.

## Design Principle

**Classes should be open for extension, but closed for modification.**

Our goal is to allow classes to be easily extended to incorporate new behavior without modifying existing code. What do we get if we accomplish this? Designs that are resilient to change and flexible enough to take on new
functionality to meet changing requirements.

Be careful when choosing the areas of code that need to be extended; applying the Open-Closed Principle EVERYWHERE is wasteful, unnecessary, and can lead to 
complex, hard to understand code.

## The Decorator Pattern defined

**The Decorator Pattern attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.**

- Decorators have the same supertype as the objects they decorate.
- You can use one or more decorators to wrap an object.
- Given that the decorator has the same supertype as the object it decorates, we can pass around a decorated object in place of the original (wrapped) object.
- **The decorator adds its own behavior either before and/or after delegating to the object it decorates to do the rest of the job.**
- Objects can be decorated at any time, so we can decorate objects dynamically at runtimewith as many decorators as we like.

## Reference

> Freeman E, Robson E, Bates B, et al. Head first design patterns[M]. " O'Reilly Media, Inc.", 2008.
