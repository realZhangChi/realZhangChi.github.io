+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = ["Abp极简教程"]
date = 2021-12-27T04:12:10Z
description = "分析CatchE项目的问题，创建领域模型，了解Abp中的实体和聚合根。"
tags = ["Tutorials", "Abp"]
title = "Abp极简教程-2 聚合"
+++

在上一篇文章中我们手动创建了**CatchE**项目并集成了Abp框架，接下来我们将向项目中添加一些实体，并介绍实体和聚合根的基本概念及用法。

## 领域

领域即是一个组织所做的事情以及其中所包含的一切，描述了**业务**规则。领域即为应用软件要解决的问题空间。

为了将我们对领域的关注点从整个软件项目中分离开来，因此创建一个类库项目名为`CatchE.Domain`，这个项目将专注于领域。

Abp项目是模块化的，在`CatchE.Domain`项目中添加Nuget包`Volo.Abp.Ddd.Domain`引用并创建`CatchEDomainModule`类。

```C#
[DependsOn(
    typeof(AbpDddDomainModule))]
public class CatchEDomainModule : AbpModule
{ }
```

## 聚合

在CatchE中，“问题”是一个关键成员，它由一个问题创建者创建，拥有标题、描述等属性，它可被指派给一个回答者。回答者可以选择回答此问题或拒绝回答，问题被拒绝回答后创建者可以指派新的回答者，问题解决后创建者可将其标记为已解决，每个问题都将拥有一个回答。问题和回答，围绕着问题为核心，组成了一部分完整的业务规则，因此在问题这一聚合下，将问题实体建模为聚合根，将回答建模为聚合内的普通实体。

### 聚合根

创建类`Issue`，继承`FullAuditedAggregateRoot<Guid>`。关于不同聚合根基类的区别，详见[Abp文档](https://docs.abp.io/en/abp/latest/Entities#base-classes-interfaces-for-audit-properties)。

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

我们采取了及早生成唯一标识的原则，将唯一标识作为参数传入构造函数中，并调用`base`构造函数为唯一标识`Id`赋值。在`Issue`对象正确实例化后，`Title`和`Description`绝不能为`null`，因此在构造函数中为它们赋值，并将它们的setter访问器访问级别设为`private`。执行构造函数时，首先利用Abp提供的`Check`实现守卫，对参数进行非空检查，然后将其赋值给属性。

显式将无参构造函数访问级别设为`private`(或`protected`)供ORM反序列化对象时使用，并可限制对象实例化的方式，阻止非法的实例化。

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

### 实体

创建实体`Answer`，并继承`AuditedEntity<Guid>`。

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

### 区别

在Abp中，聚合根和实体主要有以下区别：

1. 聚合根继承了`IHasExtraProperties`接口，集成了Abp的[对象扩展](https://docs.abp.io/en/abp/latest/Object-Extensions)功能，而实体没有；
2. 默认地，Abp将仅为聚合根创建默认实现的仓储，不会为实体创建。（[可更改](https://docs.abp.io/en/abp/latest/Entities#aggregateroot-class)）

## 总结

在这篇文章中，我们分析了CatchE项目中“问题”，并对“问题”与“答案”建模，将他们的领域知识（业务逻辑）通过代码描述出来。在下一篇文章将引入领域服务来处理实体无法完成的业务。
