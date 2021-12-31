---
title: "Abp极简教程-4 应用层及应用服务"
date: 2021-12-29T13:24:18+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "介绍应用层（Application Layer）、数据传输对象（DTO）、应用服务（Application Service），应用服务和领域服务(Domain Service)的区别，继承AutoMapper，在CatchE项目中实现应用服务。"
tags: ["Tutorials", "Abp"]
categories: ["Abp极简教程"]
---

上篇文章中，介绍了领域服务，并用领域服务实现了创建`Issue`的业务逻辑。下面介绍应用服务以及它和领域服务的区别，并在CatchE应用程序中实现创建`Issue`的功能。

## 数据传输对象

数据传输对象（DTO）通常作为应用服务的参数，由展示层调用应用服务时传入；或作为应用服务的返回值类型，在应用服务方法执行完成后将结果返回给展示层。通过数据传输对象，将展示层与领域层完全隔离开来了。数据传输对象解决了阻抗失配的问题。

{{< admonition tip "阻抗失配">}}
展示层接收的数据格式与接口返回值的数据格式一致为阻抗匹配，接口接收的参数数据格式与展示层传入参数的数据格式一致为阻抗匹配。反之，属性多余或少于所需均为阻抗失配。
{{< /admonition >}}

定义`IssueDto`和`CreateIssueDto`。

```cs
public class IssueDto : EntityDto<Guid>
{
    public string Title { get; set; }

    public string Description { get; set; }

    public Guid? AnswererId { get; set; }

    public bool IsResolved { get; set; }
}
```

```cs
public class CreateIssueDto
{
    [Required]
    public string Title { get; set; }
    
    [Required]
    public string Description { get; set; }
    
    public Guid AnswererId { get; set; }
}
```

## 应用服务

- 应用服务实现了应用程序的用例，应用服务中的每一个方法对应着应用程序中的一个用例。“提交相关信息创建`Issue`并得到创建结果”是CatchE中的一个用例，他将对应着应用服务中的一个方法。应用服务中的方法，将负责用例的任务协调。
- 应用服务是领域模型的直接客户，他将调用协调领域模型来完成一个用例。创建`Issue`的应用服务方法将会调用`Issue`、`IssueManager`、`Repository`等领域模型来完成`Issue`的创建。
- 应用服务负责控制事务以保证对模型修改的原子提交。在Abp中，自动通过`UnitOfWork`控制事务。
- 应用服务负责安全相关的操作如权限控制。

{{< admonition note >}}
应用服务的职责都是和应用程序相关的，应用服务中的“应用”二字就是“应用程序”。
{{< /admonition >}}

为了分离对应用程序的关注点，新建应用层类库项目`CatchE.Application`，添加Nuget包`Volo.Abp.Ddd.Application`引用，添加`CatchE.Domain`项目引用。为`CatchE.Application`创建Abp模块并添加模块依赖。

{{< admonition note "Contracts">}}
在Abp中，单独为应用服务接口、数据传输对象创建一个`Contracts`层是有必要的，我们会在后续教程中创建。
{{< /admonition >}}

```cs
[DependsOn(
    typeof(AbpDddApplicationModule),
    typeof(CatchEDomainModule))]
public class CatchEApplicationModule : AbpModule
{ }
```

创建`IIssueAppService`应用服务接口，并定义`CreateAsync`方法，它将接收`CreateIssueDto`参数，返回`IssueDto`。

```cs
public interface IIssueAppService : IApplicationService
{
    Task<IssueDto> CreateAsync(CreateIssueDto input);
}
```

创建`IssueAppService`应用服务，继承`IIssueAppService`接口并实现。在`CreateAsync`方法中，首先协调领域模型进行创建`Issue`的这一应用程序用例：调用领域模型中的领域服务`IssueManager`创建一个`Issue`，然后调用领域模型中的仓储`IssueRepository`将实体`Issue`保存到仓储中。然后将创建的`Issue`映射为`IssueDto`并返回。

```cs
public class IssueAppService : ApplicationService, IIssueAppService
{
    protected IssueManager IssueManager =>
        LazyServiceProvider.LazyGetRequiredService<IssueManager>();

    protected IRepository<Issue, Guid> IssueRepository =>
        LazyServiceProvider.LazyGetRequiredService<IRepository<Issue, Guid>>();

    public async Task<IssueDto> CreateAsync(CreateIssueDto input)
    {
        var issue = await IssueManager.CreateAsync(
            input.AnswererId,
            input.Title,
            input.Description);
        await IssueRepository.InsertAsync(issue);

        return ObjectMapper.Map<Issue, IssueDto>(issue);
    }
}
```

{{< admonition tip "跟随者模式">}}
基于约定，Abp中的应用服务及其接口通常以`AppService`作为后缀，并分别继承`ApplicationService`和`IApplicationService`。遵循约定对于用好Abp是至关重要的，要做一个优秀的跟随着。
{{< /admonition >}}

### 应用服务和领域服务的区别

同为服务，他们的区别即为“应用”的“领域”的区别。

应用指的是“应用程序”，应用服务是和**应用程序**相关联的服务，如应用程序用例、应用程序权限与安全、应用程序的数据库事务。

领域指的是业务，领域服务则是和**业务逻辑**相关联的服务，它实现了和应用程序用例完全无关的业务逻辑。领域服务作为领域对象的一种，和实体、仓储、领域事件等共同组成了领域模型，而领域模型则是对业务的描述。

## 使用AutoMapper

在应用服务的`CreateAsync`方法中，使用了`ObjectMapper`将实体映射为Dto。若要使用`ObjectMapper`对象映射功能，需要配置`AutoMapper。

1. 创建`CatchEApplicationAutoMapperProfile`并继承`AutoMapper.Profile`；
2. 在构造函数中创建对象映射关系；
3. 在`CatchEApplicationModule`中将注册AutoMapper配置文件到项目中。

```cs
public class CatchEApplicationAutoMapperProfile : Profile
{
    public CatchEApplicationAutoMapperProfile()
    {
        CreateMap<Issue, IssueDto>();
    }
}
```

```cs
public override void ConfigureServices(ServiceConfigurationContext context)
{
    Configure<AbpAutoMapperOptions>(options =>
    {
        options.AddMaps<CatchEApplicationModule>();
    });
}
```

`AddMaps`会对`CatchEApplicationModule`程序集内所有继承了`Profile`的类进行注册。

## 总结

这篇文章介绍了应用服务及应用层相关的概念，分析了应用服务和领域服务的区别。下一篇教程将会创建Web Api，并与CatchE启动项目集成。
