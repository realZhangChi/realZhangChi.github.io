---
title: "Abp极简教程-5 Api接口"
date: 2022-01-04T10:31:12+08:00
draft: true
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "了解Abp中的HttpApi，为CatchException项目创建HttpApi接口。"
tags: ["Tutorials", "Abp"]
categories: ["Abp极简教程"]
---

HttpApi接口承担着数据转换器的职责，它将应用层的数据传输对象，转换为便于前端使用的json格式，也将前端调用Api时传入的json格式参数，转换为应用层所需要的数据传输对象。

## 创建HttpApi

新建类库项目`CatchException.HttpApi`并引入Nuget包`Volo.Abp.AspNetCore.Mvc`，添加`CatchException.Application`项目引用。新建类`CatchExceptionHttpApiModule`并添加模块依赖。

```cs
[DependsOn(
    typeof(AbpAspNetCoreMvcModule),
    typeof(CatchExceptionApplicationModule))]
public class CatchExceptionHttpApiModule : AbpModule
{ }
```

新建`IssueController`，继承`AbpControllerBase`，并通过`IIssueAppService`实现`CreateAsync`方法，以创建`Issue`。

```cs
[Route("api/issue")]
public class IssueController : AbpControllerBase
{
    protected IIssueAppService AppService =>
        LazyServiceProvider.LazyGetRequiredService<IIssueAppService>();

    [HttpPost]
    public Task<IssueDto> CreateAsync(CreateIssueDto input)
    {
        return AppService.CreateAsync(input);
    }
}
```

## 集成

在`CatchException.HttpApi.Host`项目中引用`CatchException.HttpApi`项目，并添加`CatchExceptionHttpApiModule`模块依赖。运行项目导航至Swagger页面可看到刚添加的Api接口。

## 总结

这篇教程中介绍了HttpApi的职责并为CatchException项目创建了Api接口，下一篇文章将集成Entity Framework Core并实现仓储。
