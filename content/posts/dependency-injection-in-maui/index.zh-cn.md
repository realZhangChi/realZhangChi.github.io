+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = ["maui"]
date = 2021-10-27T10:31:05Z
description = "如何在Maui中使用依赖注入"
tags = ["dependency injection", "maui"]
title = "如何在Maui中使用依赖注入"

+++
依赖关系注入(DI)是 .NET 中的一等公民，如果熟悉 .NET 开发，对依赖注入则不会陌生。依赖关系注入是一种在类及其依赖关系之间实现控制反转(IoC)的技术，其中要反转的是获取依赖项的过程。通过依赖关系注入，分离了对象构建和对象使用的关注点，提高了代码的可读性和重用性。

.NET Multi-platform App UI 框架支持依赖关系注入软件设计模式。在 MVVM 模式中，依赖注入通常用于注册和解析视图模型，并注册和解析视图模型所依赖的服务。

## 容器

容器负责构造并注入服务，管理服务的生命周期。拥有依赖项的类，只需关注对于依赖项的使用，无需关注依赖项的创建与管理——这个过程由容器进行处理。.NET 中提供了内置的服务容器`IServiceProvider`，可以使用`IServiceProvider`来解析依赖的服务。

一般地，在应用程序启动时，将服务注册到`IServiceCollection`中，然后调用`BuildServiceProvider`扩展方法，即可得到`IServiceProvider`容器。

在 Maui 中，生成`IServiceProvider`的过程是框架自动完成的，只需要在`MauiProgram.cs`中将服务注册到`IServiceCollection`即可。

## 注册服务

在注入服务前，必须先将服务注册到容器中。Maui 内置的容器`ISeviceProvider`位于`MauiApp`中。

应用程序启动时，调用`MauiProgram.cs`中的 `CreateMauiApp`设置并构造`MauiApp`。首先调用 `CreateBuilder`创建一个构造器，通过这个构造器完成创建`MauiApp`所需的全部设置，其中包括服务注册，最终通过`Build`方法构造 `MauiApp` 实例。

```C#
public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {
    var builder = MauiApp.CreateBuilder();

    builder
    .UseMauiApp<App>()
    .ConfigureFonts(fonts =>
    {
        fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
    });

    return builder.Build();
    }
}
```

在.NET中注册服务，就是在应用程序启动时，将服务注册到 `IServiceCollection` 中。在`MauiAppBuilder`中，存在`IServiceCollection`类型的属性`Services`。因此，在 Maui 应用程序中注册服务，只需在构造`MauiApp`时将服务添加到`MauiAppBuilder`中的 `Services`中。

```C#
public static MauiApp CreateMauiApp()
{
    var builder = MauiApp.CreateBuilder();

    ...

    builder.Services.AddSingleton<MainPage>();

    return builder.Build();
}
```

在进行依赖关系注入时，需要从容器中解析服务。通过调用`IServiceCollection`的扩展方法 `BuildServiceProvider`可以构造获取 `IServiceProvider`容器实例。

调用`MauiAppBuilder`的`Build`方法获取 `MauiApp`实例时，将调用`BuildServiceProvider`，构造出`IServiceProvider`并赋值给`MauiApp`中的`Services`属性。

## 注入服务

注册服务后，可以通过容器来解析服务实例，也可将其作为依赖项进行注入。

依赖关系注入通常有构造函数注入、属性注入与方法注入三种方式。在 Maui 中，一般会使用构造方法注入依赖项，在平台代码中有时也会直接通过`MauiApp`实例来解析依赖项。

```C#
public partial class App : Application
{
    public App(MainPage mainPage)
    {
        InitializeComponent();

        MainPage = mainPage;
    }
}
```

将`MainPage`注册到容器中后，可以通过构造函数注入的方式将其作为依赖项注入。在特定平台的代码中，有时无法使用构造函数注入，这时可以直接通过容器解析依赖项。

```C#
public class MyActivity : MauiAppCompatActivity
{
    private readonly IHelloService _helloService;

    public MyActivity()
    {
        _helloService = MauiApplication.Current.Services.GetRequiredService<IHelloService>();
    }
}
```
