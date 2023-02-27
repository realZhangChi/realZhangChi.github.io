---
title: "如何在 Maui 中全局处理异常"
date: 2023-02-26T16:14:03+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在 Maui 中，结合 AOP 等技术，全局捕获 ViewModel 中产生的异常，并弹出 Toast 提示。"
tags: ["maui", "aop", "全局异常处理"]
categories: ["maui"]
---

在“[如何在Maui中使用Autofac](https://zhangchi.io/posts/use-autofac-in-maui/)”一文中介绍了，如何在 Maui 中使用 Autofac 依赖注入容器。使用 Autofac ，可以实现面向切面编程（AOP）等。在 Maui 中进行全局捕获并处理异常，可以利用 AOP 来实现。

这篇文章通过一个示例程序来展示了如何在 Maui 程序中集成 Autofac ，并基于 AOP 技术来实现全局异常处理功能。

## 引用相关 Nuget 包

新建 Maui 项目，并添加 `Autofac.Extensions.DependencyInjection`、`Autofac.Extras.DynamicProxy` Nuget 包引用。

`Autofac.Extensions.DependencyInjection` 是将 Autofac 集成到 Maui 中需要使用的包，`Autofac.Extras.DynamicProxy` 使注册到 Autofac 容器中的服务能够被拦截，以实现 AOP。

## 创建 ViewModel

在项目根目录下，创建 `ViewModels` 文件夹，并在其中创建 `MainViewModel.cs`。在 `ViewModel` 中定义名为 `RaiseException` 的方法，来模拟引发异常。

``` csharp
public class MainViewModel
{
    public virtual void RaiseException()
    {
        throw new Exception("An Exception has been throw!");
    }
}
```

{{< admonition tip "虚方法" >}}
Autofac 对类进行拦截时，将会创建一个子类并继承被拦截的类，从而实现拦截。因此被拦截类中的方法必须定义为 `virtual`，以便子类对其重写。
{{< /admonition >}}

## 创建拦截器

定义拦截器，只需定义一个类，并使其继承 `Castle.DynamicProxy.IInterceptor` 接口即可。

在 `ViewModels` 文件夹中创建 `Interceptors` 文件夹，并在其中创建 `ExceptionHandleInterceptor.cs`。

``` csharp
public class ExceptionHandleInterceptor : IInterceptor
{
    public void Intercept(IInvocation invocation)
    {
        try
        {
            invocation.Proceed();
        }
        catch (Exception e)
        {
            // TODO: Handle exception
        }
    }
}
```

`IInterceptor` 接口中定义了 `Intercept` 方法。在 `Intercept` 方法中通过 `try` 代码块将被拦截方法的调用包括起来，来捕获被拦截服务的方法调用异常，在 `catch` 代码块中对异常进行处理。

### 集成 Toaster 展示异常消息

通过在界面弹出 Toast 的方式，可以实现异常消息的用户友好提示。在 Maui Android 中集成显示 Toast 提示框的详细内容，参见“[在 Maui Android 中使用 Toaster](https://zhangchi.io/posts/use-toaster-in-maui-android/)”。

集成 Toaster 后，在 `ExceptionHandleInterceptor` 的 `Intercept` 方法的 `catch` 代码块调用 Toaster 并显示异常消息。

``` csharp
public class ExceptionHandleInterceptor : IInterceptor
{
    public void Intercept(IInvocation invocation)
    {
        try
        {
            invocation.Proceed();
        }
        catch (Exception e)
        {
            // 调用 Toaster 并显示异常消息
#if ANDROID
            Com.Hjq.Toast.Toaster.Show(e.Message);
#endif
        }
    }
}
```

## 注册服务及拦截器

编辑 `MauiProgram.cs` 文件，使 Maui 应用程序的依赖注入系统使用 Autofac 容器，并将服务及拦截器注册到容器中。关于更多在 Maui 项目中集成 Autofac 的内容，请参考“[如何在Maui中使用Autofac](https://zhangchi.io/posts/use-autofac-in-maui/)”。

``` csharp
public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {

        var builder = MauiApp.CreateBuilder();
        
        // ......

        builder.ConfigureContainer(new AutofacServiceProviderFactory((containerBuilder) =>
        {
            containerBuilder.Populate(builder.Services);

            // 注册拦截器
            containerBuilder.RegisterType<ExceptionHandleInterceptor>();

            containerBuilder.RegisterType<MainPage>();

            // 注册ViewModel服务
            containerBuilder.RegisterType<MainViewModel>()
                // 启用基于类的拦截器
                .EnableClassInterceptors()
                // 配置拦截器
                .InterceptedBy(typeof(ExceptionHandleInterceptor));

        }));

        return builder.Build();
    }
}
```

## 效果测试

更改 `MainPage.xaml.cd` 代码，将 `MainViewModel` 注入进去，并在 `OnCounterClicked` 方法中调用 ViewModel 中的 `RaiseException` 来模拟引发异常。

``` csharp
public partial class MainPage : ContentPage
{
    public MainPage(MainViewModel vm)
    {
        InitializeComponent();

        BindingContext = vm;
    }

    private async void OnCounterClicked(object sender, EventArgs e)
    {
        if (BindingContext is MainViewModel vm)
        {
            await vm.RaiseExceptionAsync();
        }
    }
}
```

运行并点击按钮，可以看到异常被拦截处理并且弹出 Toast 显示异常消息。

{{< image src="./exception-handle.gif" caption="异常消息提示" class="img-h-600">}}

## 参考内容

- [如何在Maui中使用Autofac](https://zhangchi.io/posts/use-autofac-in-maui/)
- [在 Maui Android 中使用 Toaster](https://zhangchi.io/posts/use-toaster-in-maui-android/)
- [Type Interceptors — Autofac 6.0.0 documentation](https://autofac.readthedocs.io/en/latest/advanced/interceptors.html)
