---
layout: post
title: "《CQRS Documents by Greg Young》翻译：命令和查询职责分离"
subtitle: 'CQRS Documents by Greg Young - Command and Query Responsibility Segregation'
author: "Chi"
date: 2020-06-03 10:00
header-style: text
catalog: true
tags:
  - Design Patterns
  - DDD
  - CQRS
---

这篇文章将介绍命令和查询职责分离的概念。在这里我们将研究职责分离是如何让系统体系结构变得更有效，还将分析一些应用了`CQRS`的架构的不同的特征。

## 起源

命令和查询职责分离（CQRS）最初由Bertrand Meyer提出，[维基百科](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)定义如下：

> 每个方法都应该是执行操作的命令，或者是将数据返回给调用方的查询，但不能同时是两者。 换句话说，提出问题的人不应改变他得到的答案。更正式地讲，方法仅在不产生副作用的情况下才应返回值。
It states that every method should either be a command that performs an action, or a query that returns data to the caller, but not both. In other words, Asking a question should not change the answer. More formally, methods should return a value only if they are referentially transparent and hence possess no side effects.

基本上可以归结为, 如果一个方法有返回值，则无法更改状态；如果这个方法更改状态，则返回类型必须为空。 但这可能会有一些问题，Martin Fowler对此进行了一些思考：

> Meyer喜欢完全使用命令查询分离，但是也有例外情况。**出栈**就是一个在查询时更改了状态的例子。 Meyer认为我们可以避免使用此方法，但这是一个很有用的常见用法。 因此，我更愿意在可能的情况下遵循这一原则。（Fowler）

命令和查询责任隔离最初被认为只是Meyer所提出的概念的扩展。 长期以来，它只是作为更高级别的CQS进行讨论的。 最终，在两个概念之间产生很大的混淆之后，它才认为是一种不同的模式。

命令和查询责任隔离使用了与Meyer相同的命令和查询定义，并坚持认为它们应该是纯净的。 根本的区别在于，在CQRS中，一个对象分为两个对象，一个包含命令，一个包含查询。

从系统架构的角度来看，这种模式是十分有趣的。

!["Stereotypical Architecture"](/img/in-post/2020-06-03-CommandandQueryResponsibilitySegregation/stereotypical-architecture.png)

上图展示了第一篇文章中提到的传统架构。这种架构一个典型的特征就是一个服务同时处理命令和查询。

!["Original Customer Service"](/img/in-post/2020-06-03-CommandandQueryResponsibilitySegregation/original-customer-service.png)

在其中应用CQRS的话，则会将其分为两个`Service`。

!["Customer Service After CQRS"](/img/in-post/2020-06-03-CommandandQueryResponsibilitySegregation/customer-service-after-CQRS.png)

尽管这是一个相对简单的过程，但是它将解决传统体系结构中存在的许多问题。该服务已分为两个单独的服务，即读取端和写入端，或者命令端和查询端。

这种分离体现了“命令”和“查询”有非常不同的需求。查询和命令的体系结构属性往往有很大的不同：

***一致性***

- **命令：** 比起处理各种极端情况下的最终一致性，处理具有一致性数据的事务要容易得多。

- **查询：** 大多数系统在查询的时候会保持最终一致性。

***数据存储***

- **命令：** 作为关系结构中的事务处理程序的命令端将希望以规范化的方式存储数据，类似于第三范式（3NF）。

- **查询：** 查询端以非规范化的方式来获取数据，将为了获取给定数据集而进行的表联接的数量最小化，类似于第一范式（1NF）

***可扩展性***

- **命令：** 在大多数系统中，特别是在Web系统中，命令端通常只处理很少数量的事务。因此，可扩展性并不总是很重要。

- **查询：** 在大多数系统中，尤其是Web系统中，查询端通常处理大量事务。可扩展性对查询端来说至关重要。

**不能依据同一个模型即创建用于搜索，又创建用于处理事务等不同职责的代码结构。**

## 查询端

如前所述，查询端将仅包含获取数据的方法。从传统体系结构来看，它们是返回客户端所需的DTO的所有方法。

在传统体系结构中，DTO的构建是通过映射领域域对象来处理的。DTO是与领域对象不同的模型，因此需要映射，这个过程是很痛苦的。

DTO为匹配客户端的屏幕展示，并尽可能地减少与服务端交互而创建。在有很多客户端的情况下，最好为不同的客户端建立不同的DTO。无论那种情况下，DTO模型都与为表示和处理事务而构建的领域模型完全不同。

我们能够在领域驱动设计中发现很多常见的问题：

- 在仓储上进行的很多查询方法通常包含了分页或排序信息。

- 为创建DTO，`Get`属性会公开领域内部的状态。

- 查询数据时，有时需要预先查询出其他数据（比如为了查询某个值对象相关数据，需要先查询出这些值对象所属的聚合根）。

- 加载多个聚合根以构建DTO会导致对数据模型的查询不是最优的。另外，由于DTO构建操作，可能会混淆聚合边界。

由于查询是在领域对象模型上进行操作，然后转换为数据模型，因此优化这些查询可能非常困难。开发人员需要对ORM和数据库有深入的了解，开发人员正在处理[阻抗不匹配]("https://stackoverflow.com/questions/23115988/what-is-impedance-mismatch")的问题（有关更多讨论，请参见“事件作为存储机制”）。

在应用CQRS之后，聚合边界不会被混淆，查询数据会有单独的路径，不再需要根据领域对象来映射DTO，相反，可以引入一种映射DTO的新方法。

!["The Query Side"](/img/in-post/2020-06-03-CommandandQueryResponsibilitySegregation/the-query-side.png)

这种模式下去掉了领域层，新增了一个“Thin Read Layer”，该层直接从数据库读取并构建DTO，可以通过手写`ADO.NET`、手动创建映射代码或者`ORM`进行查询。哪种选择最适合团队，很大程度上取决于团队本身以及他们最适应的条件。 最好的解决方案可能是比较居中的方案，因为ORM提供的大部分功能是用不到的，并且手动创建映射代码会浪费大量时间。 一种可能的解决方案是使用一个小的基于约定的映射实用程序。

读取层不必与数据库隔离，读取层与数据库绑定不一定是一件坏事。使用存储过程进行读取也不一定不好，这也是取决于团队和系统的非功能性要求。

读取层虽然维护起来可能很繁琐，但却不是复杂的代码。分离的读取层的一个好处是它不会遭受阻抗失配的困扰。它直接连接到数据模型，这可以使查询更易于优化。编写查询端代码的开发人员也不需要了解域模型，也不需要使用任何ORM工具，他们只需要了解数据模型。

实际上，读取层的分离或者说从领域中分离出来查询操作，也是领域中的一个特殊组成部分。

## 命令端

总的来说，命令端仍然与"传统架构"非常相似。下图中的接口看起来应该和之前讨论过的架构基本相同。主要的区别在于，它现在有一个行为上的契约，而不是以数据为中心的契约，这是实际使用领域驱动设计所需要的，而且它的读取已经被分离出来。

!["The Command Side"](/img/in-post/2020-06-03-CommandandQueryResponsibilitySegregation/the-command-side.png)

在"传统架构"中，领域同时处理命令和查询，这在领域内造成了许多问题：

- 在仓储上进行的很多查询方法通常包含了分页或排序信息。

- 为创建DTO，`Get`属性会公开领域内部的状态。

- 查询数据时，有时需要预先查询出其他数据（比如为了查询某个值对象相关数据，需要先查询出这些值对象所属的聚合根）。

- 加载多个聚合根以构建DTO会导致对数据模型的查询不是最优的。另外，由于DTO构建操作，可能会混淆聚合边界。

一旦查询被分离出来，领域将专注于命令的执行，这些问题自然也随风而去。领域对象突然间不再需要暴露内部状态，仓储除了`GetById`方法之外，几乎没有任何查询方法，而且可以将更多的行为重点放在Aggregate边界上。

这种改变与原来的结构相比，开发成本更低，甚至没有成本。在许多情况下，分离实际上会降低开发成本，因为在读取层中对查询的优化比在领域模型中实现时更简单。该架构在与领域模型一起工作时，因为查询分离了，所以读取端的开发人员不用深入理解领域概念，这也能让开发成本降低。在最坏的情况下，开发成本应该是等价的。这种改变真正做到了责任的转移，甚至读取端仍然可以使用领域。

通过应用CQRS，已将读取和写入的概念分开。这确实引出了一个问题，即两者是否应该存在于读取相同的数据模型中，或者可以将它们视为两个集成系统来对待，下图说明了这一概念。为了以一致或最终一致的方式保持同步，多个数据源之间存在许多众所周知的集成模式。这两个不同的数据源允许针对当前任务优化数据模型。例如，可以针对读取端进行第一范式建模，而针对命令端进行第三范式建模。

多个数据源的集成模式的选择非常重要，因为模型之间的转换和同步可能会成为一项非常昂贵的工作。最适合的模式是引入事件，事件是一种众所周知的集成模式，为模型同步提供了最好的机制。

!["Separated Data Models with CQRS"](/img/in-post/2020-06-03-CommandandQueryResponsibilitySegregation/separated-data-models-with-CQRS.png)

## 引用

> [CQRS Documents by Greg Young](https://cqrs.files.wordpress.com/2010/11/cqrs_documents.pdf)
> [What is Impedance Mismatch?](https://stackoverflow.com/questions/23115988/what-is-impedance-mismatch)
> [Command–query separation](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)
