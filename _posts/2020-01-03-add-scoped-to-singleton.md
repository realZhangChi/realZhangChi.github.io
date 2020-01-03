---
layout: post
title: "将生命周期注册为Scoped的依赖项，注入到生命周期注册为Singleton的类中"
subtitle: 'Register the lifecycle as a Scoped dependency and inject it into the class registered as Singleton'
author: "Chi"
date: 2020-01-03 14:00
header-style: text
catalog: false
tags:
  - ASP.NET Core
  - Dependency Injection
---

最近在项目中，结合Quartz.NET和`IHostedService`实现了.NET Core控制台的计划任务。在这个项目中，需要注入`DbContext`来进行数据库连接与操作。`IServiceCollection`的扩展方法`AddDbContext`是将`DbContext`注入为`Scoped`类型的生命周期的，而Quartz.NET要求我们将`IJobFactory`和`ISchedulerFactory`注入为`Singleton`类型的。当我们在`Singleton`类型的类中直接注入`Scoped`类型的依赖项时，会抛出错误"不能在Singleton中使用Scoped类型的服务"：

``` Powershell
Cannot consume scoped service 'XXX' from singleton 'XXX'.
```

## 解决方案

在需要使用`Scoped`类型依赖项的类中，不直接注入此依赖项，而是注入`IServiceScopeFactory`依赖项：

``` C#
public class MySingletonService : IMySingletonService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public MySingletonService(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }
}
```

在需要使用`Scoped`类型依赖项的方法中，通过IServiceScopeFactory的实例解析此依赖项即可：

```C#
public class MySingletonService : IMySingletonService
{
    // Other code

    public void Scoped()
    {
        using var scope = _scopeFactory.CreateScope();
        var ctx = scope.ServiceProvider.GetRequiredService<MyDbContext>();

        // Other code
    }
}
```
