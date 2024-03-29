+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = ["Abp极简教程"]
date = 2021-12-23T04:12:10Z
description = "从手动创建项目CatchException开始，添加Abp应用、模块来集成Abp框架。"
tags = ["Tutorials", "Abp"]
title = "Abp极简教程1 应用和模块"

+++
Abp是一个基于.NET的开源应用程序框架，它遵循最佳实践和约定，根据DDD模式进行设计和开发，并提供了强大的基础设施和完整的架构。

Abp提供了项目启动模板，它依据DDD模式进行分层，并预先配置了常用的模块。启动模板中反映着领域驱动设计、最佳实践等多种概念，其中任一项都值得单独讨论。对于初学者而言，启动模板中的大量知识如潮水般瞬间涌入脑海，造成知识过载，无法聚焦当前真正要学习的知识。

本系列教程，将结合一个问答网站**CatchException**示例项目，从简单的.NET Web Api应用程序开始，搭建起基于Abp的Web应用程序框架，并逐步深入细节，旨在以一种缓和的学习曲线帮助初学者快速入门，理解Abp启动模板的分层项目。

{{< admonition tip "关注代码" >}}
教程中会对必要的领域驱动设计概念进行简单的描述，如果在阅读本教程时对其感到困惑，那么暂时不要深入领域驱动设计的细节，请专注于代码及代码的设计思路。
{{< /admonition >}}

## 创建应用

通过VS创建一个名为`CatchException`Web Api应用，选择ASP.NET Core Web Api项目模板，或通过CLI执行`dotnet new webapi -n CatchException`。运行后将访问至Swagger页面。

## 集成Abp

首先需要添加`Volo.Abp.Autofac`和`Volo.Abp.AspNetCore.Mvc` Nuget包引用至项目中以集成Abp框架。

### Abp应用

Abp框架中定义了`IAbpApplication`应用，这也是Abp项目的入口点，Abp应用包含了启动模块及其依赖。在项目启动时需要将Abp应用注册到依赖注入系统中去，并指定启动模块。Abp定义了`AddApplication`泛型扩展方法，它将Abp应用注册为单例，方法的泛型参数指定了启动模块。

通过`WebApplicationBuilder`编译得到`WebApplication`后，调用Abp定义的扩展方法`InitializeApplication`，他将初始化Abp应用，根据模块的依赖关系初始化启动模块及其依赖的模块。

```cs
var builder = WebApplication.CreateBuilder(args);
builder.Host
    .UseAutofac();
builder.Services.AddApplication<CatchExceptionModule>(
    options =>
    {
        options.Services.ReplaceConfiguration(builder.Configuration);
    });
var app = builder.Build();
app.InitializeApplication();
await app.RunAsync();
```

### 模块

Abp设计为模块化的应用程序框架，每一个模块都应定义一个继承自`AbpModule`的类，并以`Module`后缀作为类名。不同的模块间会存在依赖关系，模块的依赖关系通过`DependsOn`特性来定义。每个C#项目只应定义一个模块。

在`ConfigureServices`方法中，可以将依赖项注册到依赖注入系统中。在Abp中，通过约定大于配置的方式进行依赖项注册，项目代码通常无需在这里手动注册。`ConfigureServices`方法将在实例化Abp应用的时候调用以进行依赖项注册。

初始化Abp应用时，将会按照依赖顺序初始化所有的模块。初始化启动项模块时将会调用`OnApplicationInitialization`方法，通常在这个方法中会构建中间件管道。

```cs
[DependsOn(
    typeof(AbpAutofacModule),
    typeof(AbpAspNetCoreMvcModule))]
public class CatchExceptionModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    { }

    public override void OnApplicationInitialization(ApplicationInitializationContext context)
    {
        var app = context.GetApplicationBuilder();
        var env = context.GetEnvironment();

        app.UseRouting();
        app.UseConfiguredEndpoints();
    }
}
```

### Swagger

添加Nuget包引用`Volo.Abp.Swashbuckle`及其模块依赖。Abp对Swagger进行了一些功能上的完善，比如防止跨站点请求伪造攻击，因此在项目中需要使用`Volo.Abp.Swashbuckle`。`Volo.Abp.Swashbuckle`自定义了swagger所需的js文件，他位于`Volo.Abp.Swashbuckle`模块的wwwroot文件夹中，因此需要通过`UseStaticFiles`扩展方法添加静态文件中间件支持。

```cs
[DependsOn(
    typeof(AbpAutofacModule),
    typeof(AbpAspNetCoreMvcModule),
    typeof(AbpSwashbuckleModule))]
public class CatchExceptionModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        ConfigureSwaggerServices(context);
    }

    public override void OnApplicationInitialization(ApplicationInitializationContext context)
    {
        var app = context.GetApplicationBuilder();
        var env = context.GetEnvironment();

        app.UseRouting();
        if (env.IsDevelopment())
        {
            app.UseSwagger();
            app.UseAbpSwaggerUI(c =>
            {
                c.SwaggerEndpoint("/swagger/v1/swagger.json", "CatchException API");
            });
        }
        app.UseConfiguredEndpoints();
    }


    private static void ConfigureSwaggerServices(ServiceConfigurationContext context)
    {
        context.Services.AddAbpSwaggerGen();
    }
}
```

### 日志

添加Nuget包引用`Serilog.AspNetCore`、`Serilog.Sinks.Async`到项目中。

在应用程序启动时，首先创建一个Serilog日志记录器，然后将构建并运行Web应用的操作通过`try`块包括起来捕获异常，在`catch`块中记录启动异常日志，在`finally`块中重置Serilog日志记录器。上述操作针对启动过程进行了日志记录，若要使应用通过Serilog记录日志，还需要`UseSerilog`扩展方法注册Serilog日志服务。更改Program.cs。

```cs
try
{
    Log.Logger = new LoggerConfiguration()
#if DEBUG
        .MinimumLevel.Debug()
#else
        .MinimumLevel.Information()
#endif
        .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
        .Enrich.FromLogContext()
        .WriteTo.Async(c => c.File("Logs/logs-.txt", rollingInterval: RollingInterval.Day))
#if DEBUG
        .WriteTo.Async(c => c.Console())
#endif
        .CreateLogger();

    var builder = WebApplication.CreateBuilder(args);
    builder.Host
        .UseAutofac()
        .UseSerilog();
    builder.Services.AddApplication<CatchExceptionModule>(
        options =>
        {
            options.Services.ReplaceConfiguration(builder.Configuration);
        });
    var app = builder.Build();
    app.InitializeApplication();
    await app.RunAsync();
    return 0;
}
catch (Exception ex)
{
    Log.Fatal(ex, "Host terminated unexpectedly!");
    return 1;
}
finally
{
    Log.CloseAndFlush();
}
```

### 启动

启动应用此时应导航到Swagger页面并可调用`WeatherForecast`接口获取数据。

## 总结

这篇文章展示了如何从ASP.NET Core Web Api模板开始，手动集成Abp框架并将项目模块化，以当前项目作为启动模块创建并运行Abp应用。这里简单介绍了Abp应用及Abp模块，后续文章将逐步介绍Abp中的其他概念及用法。
