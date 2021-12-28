+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = ["Abp极简教程"]
date = 2021-12-27T04:12:10Z
description = "分析CatchE项目要解决的问题，创建领域模型，了解Abp中的实体和聚合根。"
tags = ["Tutorials", "Abp"]
title = "Abp极简教程-2 实体、聚合根"
+++

在上一篇文章中我们手动创建了**CatchE**项目并集成了Abp框架，接下来我们将向项目中添加一些实体，并介绍实体和聚合根的基本概念及用法。

## 领域

领域即是一个组织所做的事情以及其中所包含的一切，描述了**业务**规则。领域即为应用软件要解决的问题空间。

问答平台的需求大致可以描述为以下几点：

1. 提问者创建一个问题
2. 问题拥有标题、描述、是否解决等属性
3. 提问者可以请求一个回答者来回答问题
4. 问题被拒绝回答后创建者可以请求新的回答者来回答问题
5. 问题解决后创建者可将其标记为已解决
6. 每个问题都将拥有一个回答

CatchE项目将实现上述需求。很显然，为实现上述需求，需要其他功能的支持，比如身份验证、权限控制等，但是上述需求将是我们业务成功的关键因素，因此上述需求将是我们的核心域，其他功能是支撑子域或通用子域。开发核心域的解决方案是一种关键性业务投入，CatchE项目即为我们实现核心域功能的解决方案。

>如果你对子域、支撑子域、通用子域知之甚少，那么请不要考虑他们，只需要将关注点集中在上述需求和如何实现以上需求即可。

为了将领域的关注点从实现细节如数据库、Web Api等分离开来，因此创建一个类库项目名为`CatchE.Domain`。领域层将专注于业务逻辑。

Abp项目是模块化的，在`CatchE.Domain`项目中添加Nuget包`Volo.Abp.Ddd.Domain`引用并创建`CatchEDomainModule`类。

```C#
[DependsOn(
    typeof(AbpDddDomainModule))]
public class CatchEDomainModule : AbpModule
{ }
```

## 聚合

很显然，在我们要解决的问题中，存在提问者、问题、回答者、答案四个关键模型，我们将暂时不考虑提问者，因为任何一个用户都可以提问问题。将问题、答案、回答者命名为：`Issue`、`Answer`、`Answerer`。

对于`Answer`，他将是`Issue`的一部分，无法离开`Issue`单独存在，此外，`Issue`和`Answer`还拥有事务一致性，因此将`Issue`和`Answer`划分为一个聚合，并将`Issue`建模为聚合根。

`Answerer`和`Issue`聚合不存在强耦合关系，可以独立地维护`Answerer`的状态，因此以`Answerer`为聚合根建模为另一个聚合。

### Issue聚合

#### Issue聚合根

创建聚合根`Issue`，继承`FullAuditedAggregateRoot<Guid>`。关于不同聚合根基类的区别，详见[Abp文档](https://docs.abp.io/en/abp/latest/Entities#base-classes-interfaces-for-audit-properties)。

为聚合根`Issue`添加属性`Title`、`Description`、`AnswererId`和`IsResolved`。

显式将无参构造函数访问级别设为`private`(或`protected`)供ORM反序列化对象时使用，并可限制对象实例化的方式，阻止非法的实例化。

创建以唯一标识、`title`、和`description`作为参数的构造函数。基于及早生成唯一标识的原则，将唯一标识作为参数传入构造函数中，并调用`base`为唯一标识`Id`赋值。在`Issue`对象正确实例化后，`Title`和`Description`绝不能为`null`，因此在构造函数中为它们赋值，并将它们的set访问器访问级别设为`private`。在构造函数中，首先利用Abp提供的`Check`实现守卫，对参数进行非空检查，然后将其赋值给属性。

值得一提的是，`Issue`只能通过有参的构造函数来实例化，将创建`Issue`所需的所有信息都传递给构造函数，构造函数保证满足所有满足所有固定规则，使得创建操作是原子的。这也是构造函数这一机制的本意，对象的创建应通过构造函数来进行，构造函数保证对象的合法性。很多代码中，首先调用无参构造函数，然后通过属性的set访问器更改状态，从而完成对象的创建，创建对象的职责从构造函数转移到了编码人员，这种行为在一个长期迭代的软件系统中是相当危险的。

```C#
public class Issue : FullAuditedAggregateRoot<Guid>
{
    public string Title { get; private set; }

    public string Description { get; private set; }

    public Guid? AnswererId { get; private set; }

    public bool IsResolved { get; private set; }

    private Issue()
    {

    }

    public Issue(
        Guid id,
        string title,
        string description) : base(id)
    {
        Title = Check.NotNullOrWhiteSpace(title, nameof(title));
        Description = Check.NotNullOrWhiteSpace(description, nameof(description));
    }
}
```

问题可以指派给一个回答者，因此在`Issue`中添加方法：

```C#
public Issue AssignTo(Guid answererId)
{
    if (AnswererId.HasValue)
    {
        throw new BusinessException(message: "不可重复指派");
    }

    AnswererId = answererId;
    return this;
}
```

上文描述问题的业务规则时，提到“*问题被拒绝回答后创建者可以指派新的回答者*”，这一描述隐含着另一条规则：“只有问题被拒绝回答后**才可以**指派新的回答者”。因此在`AssignTo`方法的卫语句中对`AnswererId`进行判断并处理。

问题可以被取消指派（回答者拒绝回答问题）：

```C#
public Issue CancelAssign()
{
    AnswererId = null;

    return this;
}
```

问题最终将会被解决：

```C#
public Issue Resolved()
{
    IsResolved = true;

    return this;
}
```

#### Answer实体

回答者在答案被采纳前可以修改回答。创建实体`Answer`，并继承`AuditedEntity<Guid>`。

```C#
public class Answer : AuditedEntity<Guid>
{
    public Guid IssueId { get; init; }

    public string Content { get; private set; }

    public virtual Issue Issue { get; set; }

    private Answer()
    {

    }

    public Answer(
        Guid id,
        Guid issueId,
        string content) : base(id)
    {
        IssueId = issueId;
        Content = Check.NotNullOrWhiteSpace(content, nameof(content));
    }
}
```

回答者可以更改答案。

```C#
public Answer Change(string content)
{
    Content = Check.NotNullOrWhiteSpace(content, nameof(content));

    return this;
}
```

将`Answer`作为导航属性添加到`Issue`中。

```C#
public virtual Answer Answer { get; set; }
```

### Answerer聚合

创建类`Answerer`，继承`FullAuditedAggregateRoot<Guid>`。随着业务的深入，将为`Answerer`扩展更多的功能，比如设置赞赏码等。

```C#
public class Answerer : FullAuditedAggregateRoot<Guid>
{
    public Guid IdentityUserId { get; set; }

    public string Name { get; set; }

    public Answerer(
        Guid id,
        Guid identityUserId,
        string name) : base(id)
    {
        IdentityUserId = identityUserId;
        Name = name;
    }
}
```

### AggregateRoot和Entity的区别

在Abp中，聚合根和实体主要有以下区别：

1. 聚合根继承了`IHasExtraProperties`接口，集成了Abp的[对象扩展](https://docs.abp.io/en/abp/latest/Object-Extensions)功能，而实体没有；
2. 默认地，Abp将仅为聚合根创建默认实现的仓储，不会为实体创建。（[可更改](https://docs.abp.io/en/abp/latest/Entities#aggregateroot-class)）

## 总结

在这篇文章中，我们分析了CatchE项目的业务需求，得到了核心域，识别出核心域的两个聚合并对他们进行建模。在下一篇文章将引入领域服务来处理实体无法完成的业务。
