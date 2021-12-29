---
title: "Abp极简教程-4 应用服务"
date: 2021-12-29T13:24:18+08:00
draft: true
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "介绍应用服务，应用服务和领域服务的区别，在CatchE项目中实现应用服务。"
tags: ["Tutorials", "Abp"]
categories: ["Abp极简教程"]
---

上篇文章中，介绍了领域服务，并用领域服务实现了创建`Issue`的业务逻辑。下面介绍应用服务以及它和领域服务的区别，并在CatchE应用程序中实现创建`Issue`的功能。

## 应用服务

- 应用服务实现了应用程序的用例，应用服务中的每一个方法对应着应用程序中的一个用例。“提交相关信息创建`Issue`并得到创建结果”是CatchE中的一个用例，他将对应着应用服务中的一个方法。应用服务中的方法，将负责用例的任务协调。
- 应用服务是领域模型的直接客户，他将调用协调领域模型来完成一个用例。创建`Issue`的应用服务方法将会调用`Issue`、`IssueManager`、`Repository`等领域模型来完成`Issue`的创建。
- 应用服务负责控制事务以保证对模型修改的原子提交。在Abp中，自动通过`UnitOfWork`控制事务。
- 应用服务负责安全相关的操作如权限控制。

{{< admonition note >}}
应用服务的职责都是和应用程序相关的，应用服务中的“应用”二字就是“应用程序”。
{{< /admonition >}}

### 创建应用层

为了分离对应用程序的关注点，新建应用层类库项目`CatchE.Application`，添加Nuget包`Volo.Abp.Ddd.Application`引用，添加`CatchE.Domain`项目引用。为`CatchE.Application`创建Abp模块并添加模块依赖。

```cs
[DependsOn(
    typeof(AbpDddApplicationModule),
    typeof(CatchEDomainModule))]
public class CatchEApplicationModule : AbpModule
{ }
```
