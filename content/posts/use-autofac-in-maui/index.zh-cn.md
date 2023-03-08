---
title: "如何在Maui中使用Autofac"
date: 2023-02-21T13:45:20+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在Maui中使用Autofac容器来管理对象的生命周期"
tags: ["dependency injection", "maui", "autofac"]
categories: ["maui"]
---

Autofac是一个开源的控制反转容器，通过将.NET程序的控制反转容器替换为Autofac，可以实现例如属性注入、面向切面编程等功能。

## 引用Autofac

从NuGet引用 `Autofac.Extensions.DependencyInjection`包。

## 使用Autofac容器

打开`MauiProgram.cs`文件，在**所有代码最后**、`return builder.Build();`**之前**，通过`ConfigureContainer`来使用`AutofacServiceProviderFactory`来构建使用Autofac容器：

``` csharp
public static MauiApp CreateMauiApp()
{
    var builder = MauiApp.CreateBuilder();
    builder
        .UseMauiApp<App>()
        .ConfigureFonts(fonts =>
        {
            fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
            fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
        });
#if DEBUG
    builder.Logging.AddDebug();
#endif

    // 添加以下代码
    builder.ConfigureContainer(new AutofacServiceProviderFactory((containerBuilder) =>
    {
        // Once you've registered everything in the ServiceCollection, call
        // Populate to bring those registrations into Autofac. This is
        // just like a foreach over the list of things in the collection
        // to add them to Autofac.
        containerBuilder.Populate(builder.Services);

        // Make your Autofac registrations. Order is important!
        // If you make them BEFORE you call Populate, then the
        // registrations in the ServiceCollection will override Autofac
        // registrations; if you make them AFTER Populate, the Autofac
        // registrations will override. You can make registrations
        // before or after Populate, however you choose.
        containerBuilder.RegisterType<MainPage>();
        containerBuilder.RegisterType<MainViewModel>();
    }));

    return builder.Build();
}

```

在上述代码中，通过调用`Populate`方法，将Maui在`ServiceCollection`中的服务注册，配置到了Autofac容器中，然后通过`RegisterType`将项目中的服务注册到Autofac容器中。

{{< admonition tip "关注代码顺序" >}}
上述步骤中的代码顺序至关重要，他将影响服务在容器中的注册，详情参见[Autofac文档](https://autofac.readthedocs.io/en/latest/integration/netcore.html)。
{{< /admonition >}}

## 解析依赖项

通过上述步骤将依赖注入容器替换为Autofac，将不会影响在Maui中解析依赖项的方式。正如[《如何在Maui中使用依赖注入》](https://zhangchi.io/posts/dependency-injection-in-maui/)一文中所介绍的，可以通过构造函数来解析依赖项。

``` csharp
public partial class App : Application
{
    public App(MainPage mainPage)
    {
        InitializeComponent();

        MainPage = mainPage;
    }
}
```

## 推荐内容

- [《如何在Maui中使用依赖注入》](https://zhangchi.io/posts/dependency-injection-in-maui/)

## 源码获取

扫描下方二维码，关注公众号**捕获异常**，回复 **maui** 获取源码。
