---
title: "Abp极简教程-3 领域服务"
date: 2021-12-28T09:41:38+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在CatchE项目中引入领域服务，来处理业务逻辑。"
tags: ["Tutorials", "Abp"]
categories: ["Abp极简教程"]
---

在上一篇教程中，分析了CatchE项目的业务逻辑，得到了核心域，并对`Issue`、`Answerer`聚合建模。这篇文章中将创建领域服务，来完成提问`Issue`的功能。

## 领域服务

{{< admonition note "DDD" >}}
领域服务处理业务规则。实现业务逻辑时，首先应考虑实体。如果实体无法**独立**实现业务，或者某个操作过程不属于实体的职责时，便应该将其放在领域服务中。
{{< /admonition >}}

在CatchE中，提出一个`Issue`时，应选择一个`Answerer`回答问题。若不存在`Answerer`与选中的用户匹配，则根据选中用户进行创建。

通过构造函数创建`Issue`时，无法得知回答者是否存在，也无法创建`Answerer`。对`Answerer`的处理不是`Issue`的职责,因此创建领域服务来实现创建`Issue`的业务逻辑。

创建类`IssueManager`，继承`DomainService`。Abp框架遵循了约定大于配置的概念，继承了`DomainService`的类，Abp会将其视为领域服务并注册到依赖注入容器中。

```cs
public class IssueManager : DomainService
{ }
```

需要通过仓储来判断`Answerer`是否存在，通过`DomainService`的`LazyServiceProvider`以懒加载的方式注入`IRepository<Answerer, Guid>`。

{{< admonition >}}
仓储及仓储的实现，将在后面的文章中介绍。
{{< /admonition >}}

```cs
protected IRepository<Answerer, Guid> AnswererRepository =>
        LazyServiceProvider.LazyGetRequiredService<IRepository<Answerer, Guid>>();
```

创建方法`CreateAsync`，方法中首先判断`Answerer`是否存在，若不存在则新建。新建`Answerer`时，需要确保`IdentityUserId`这一用户是存在的，因此还需注入`IRepository<IdentityUser, Guid>`仓储。完成`Answerer`的处理后，创建`Issue`并将其指派给`Answerer`。完整代码如下。

```cs
public class IssueManager : DomainService
{
    protected IRepository<Answerer, Guid> AnswererRepository =>
        LazyServiceProvider.LazyGetRequiredService<IRepository<Answerer, Guid>>();

    protected IRepository<IdentityUser, Guid> IdentityUserRepository =>
        LazyServiceProvider.LazyGetRequiredService<IRepository<IdentityUser, Guid>>();

    public async Task<Issue> CreateAsync(
        Guid answererIdentityUserId,
        string title,
        string description)
    {
        var answerer = await AnswererRepository
            .SingleOrDefaultAsync(a => a.IdentityUserId == answererIdentityUserId);
        if (answerer is null)
        {
            var userExists = await IdentityUserRepository
                .AnyAsync(u => u.Id == answererIdentityUserId);
            if (!userExists)
            {
                throw new UserFriendlyException("用户不存在");
            }

            answerer = await AnswererRepository.InsertAsync(
                new Answerer(
                    GuidGenerator.Create(),
                    answererIdentityUserId));
        }

        var issue = new Issue(
            GuidGenerator.Create(),
            title,
            description);
        issue.AssignTo(answerer);

        return issue;
    }
}
```

{{< admonition tips "唯一标识" >}}
实体的唯一标识应在实例化的时候由构造函数的调用方提供。通过`GuidGenerator.Create()`可生成顺序GUID。
{{< /admonition >}}

## 总结

这篇文章中介绍了领域服务的概念，并通过分析创建`Issue`业务逻辑，完成了对领域服务的建模。下一篇文章将会分析CatchE应用程序的用例，并实现应用服务。
