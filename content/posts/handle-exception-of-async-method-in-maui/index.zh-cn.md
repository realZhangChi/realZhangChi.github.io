---
title: "如何在 Maui 中全局处理异常（异步方法）"
date: 2023-02-27T15:40:52+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在 Maui 中，结合 AOP 等技术，全局捕获 ViewModel 中异步方法产生的异常，并弹出 Toast 提示。"
tags: ["maui", "aop", "autofac"]
categories: ["maui"]
---

在“[如何在 Maui 中全局处理异常](https://zhangchi.io/posts/handle-exception-in-maui/)”一文中介绍了通过 Autofac 动态代理，对 `ViewModel` 进行拦截，全局捕获 `ViewModel` 中的异常，并使用 Toaster 弹出异常消息提示。然而，Autofac 中的动态代理只支持对同步方法进行拦截。本文将继续介绍如何实现对异步方法的代理拦截。

## 先决条件

[如何在 Maui 中全局处理异常](https://zhangchi.io/posts/handle-exception-in-maui/)

## Castle.Core.AsyncInterceptor

[Castle.Core.AsyncInterceptor](https://github.com/JSkimming/Castle.Core.AsyncInterceptor) 是一个简化异步方法拦截的库，通过引入 Castle.Core.AsyncInterceptor ，可以在 Maui 中轻松实现对异步方法的拦截。

## 更新项目

调整在“[如何在 Maui 中全局处理异常](https://zhangchi.io/posts/handle-exception-in-maui/)”中创建的项目，更改ViewModel中的 `RaiseException` 方法为异步的 `RaiseExceptionAsync` 方法，并返回 `Task`。

``` csharp
public virtual Task RaiseExceptionAsync()
{
    throw new Exception("An Exception has been throw!");
}
```

更改 `MainPage.xaml.cs`中的 `OnCounterClicked` 方法，为方法签名加入 `async` 关键字，将 `RaiseException` 方法调用改为异步并等待。

``` csharp
private async void OnCounterClicked(object sender, EventArgs e)
{
    if (BindingContext is MainViewModel vm)
    {
        await vm.RaiseExceptionAsync();
    }
}
```

## 引入 Castle.Core.AsyncInterceptor

通过 CLI 或 Nuget 包管理器添加对 Castle.Core.AsyncInterceptor 的引用。

## 实现异步拦截器

Castle.Core.AsyncInterceptor 定义了 `IAsyncInterceptor` 接口，创建继承 `IAsyncInterceptor` 接口的类即可实现异步拦截器。

在 `ViewModels/Interceptors` 路径下，创建类 `ExceptionHandleAsyncInterceptor`，继承 `IAsyncInterceptor` 接口并实现接口中定义的方法。

``` csharp
public class ExceptionHandleAsyncInterceptor : IAsyncInterceptor
{
    public void InterceptSynchronous(IInvocation invocation)
    {
        try
        {
            invocation.Proceed();
        }
        catch (Exception e)
        {
#if ANDROID
            Com.Hjq.Toast.Toaster.Show(e.Message);
#endif
        }
    }

    public void InterceptAsynchronous(IInvocation invocation)
    {
        invocation.ReturnValue = InternalInterceptAsynchronous(invocation);
    }

    private async Task InternalInterceptAsynchronous(IInvocation invocation)
    {

        try
        {
            invocation.Proceed();
            var task = (Task)invocation.ReturnValue;
            await task;
        }
        catch (Exception e)
        {
#if ANDROID
            Com.Hjq.Toast.Toaster.Show(e.Message);
#endif
        }
    }

    public void InterceptAsynchronous<TResult>(IInvocation invocation)
    {
        invocation.ReturnValue = InternalInterceptAsynchronous<TResult>(invocation);
    }

    private async Task<TResult?> InternalInterceptAsynchronous<TResult>(IInvocation invocation)
    {
        try
        {
            invocation.Proceed();
            var task = (Task<TResult?>)invocation.ReturnValue;
            TResult? result = await task;

            return result;
        }
        catch (Exception e)
        {
#if ANDROID
            Com.Hjq.Toast.Toaster.Show(e.Message);
#endif
            return default;
        }
    }
}
```

`IAsyncInterceptor` 中定义了 `InterceptSynchronous`、`InterceptAsynchronous`、`InterceptAsynchronous<TResult>` 三个方法。`InterceptSynchronous` 用于实现对同步方法的拦截，`InterceptAsynchronous` 用于实现对返回值为 `Task` 的异步方法的拦截，`InterceptAsynchronous<TResult>` 用于实现对返回值为泛型的 `Task<TResult>` 异步方法的拦截。关于 Castle.Core.AsyncInterceptor 的更多用法，参见其 [Github 仓库](https://github.com/JSkimming/Castle.Core.AsyncInterceptor)。

## 注册使用异步拦截器

在 `MainProgram.cs` 文件中注册新建的异步拦截器。

``` csharp
containerBuilder.RegisterType<ExceptionHandleAsyncInterceptor>();
```

{{< admonition tip >}}
不要将 `MainViewModel` 服务注册代码中的 `InterceptedBy` 方法参数更改为 `typeof(ExceptionHandleAsyncInterceptor)`。
对 `MainViewModel` 的拦截行为仍将使用基于 `IInterceptor` 的拦截器。对于异步方法的支持将通过在基于 `IInterceptor` 的拦截器中来调用异步拦截器来实现。
{{< /admonition >}}

修改原来同步方法使用的 `ExceptionHandleInterceptor` 拦截器，将异步拦截器注入其中，并将对方法的拦截操作交给异步拦截器。

``` csharp
public class ExceptionHandleInterceptor : IInterceptor
{
    private readonly ExceptionHandleAsyncInterceptor _asyncInterceptor;

    public ExceptionHandleInterceptor(ExceptionHandleAsyncInterceptor asyncInterceptor)
    {
        _asyncInterceptor = asyncInterceptor;
    }

    public void Intercept(IInvocation invocation)
    {
        _asyncInterceptor.ToInterceptor().Intercept(invocation);
    }
}
```

## 效果测试

启动项目并点击页面上的按钮，将会看到异常被捕获并通过 `Toaster` 展示出来。

{{< image src="../handle-exception-in-maui/exception-handle.gif" caption="异常消息提示" class="img-h-600">}}

## 参考内容

- [如何在Maui中使用Autofac](https://zhangchi.io/posts/use-autofac-in-maui/)
- [在 Maui Android 中使用 Toaster](https://zhangchi.io/posts/use-toaster-in-maui-android/)
- [如何在 Maui 中全局处理异常](https://zhangchi.io/posts/handle-exception-in-maui/)
- [Type Interceptors — Autofac 6.0.0 documentation](https://autofac.readthedocs.io/en/latest/advanced/interceptors.html)
- [Castle.Core.AsyncInterceptor](https://github.com/JSkimming/Castle.Core.AsyncInterceptor)

## 源码获取

扫描下方二维码，关注公众号**捕获异常**，回复 **maui** 获取源码。
