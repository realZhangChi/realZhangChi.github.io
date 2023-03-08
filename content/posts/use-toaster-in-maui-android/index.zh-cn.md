---
title: "在 Maui Android 中使用 Toaster"
date: 2023-02-23T14:22:37+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "如何在 Maui 项目中使用 Toaster"
tags: ["maui", "binding library"]
categories: ["maui"]
---

## Toaster Android Sdk

Toaster 是一个开源的 Android 吐司框架，原名 ToastUtils ，已更名为 **Toaster** 。
Toaster 项目在 github 上开源，地址：[https://github.com/getActivity/Toaster](https://github.com/getActivity/Toaster).

## Toaster 绑定库

在 Maui 项目中使用原生 SDK 非常简单，只需要创建绑定类库即可。在 [MauiBinding](https://github.com/realZhangChi/MauiBinding) 项目中已为 Toaster 创建好了绑定类库，并可在 [Nuget](https://www.nuget.org/packages/Chi.MauiBinding.Android.Toaster/) 上下载。通过 Nuget 包管理器或 CLI 安装 `Chi.MauiBinding.Android.Toaster` 即可使用 Toaster。

## 使用 Toaster

### 初始化 Toaster

根据 [Toaster 文档](https://github.com/getActivity/Toaster#%E5%88%9D%E5%A7%8B%E5%8C%96%E6%A1%86%E6%9E%B6)，要使用 Toaster ，首先需要进行初始化配置。

在 Maui 项目的 `Platforms/Android/MainApplication.cs` 文件中，重写 `OnCreate` 方法，并调用 `Toaster.Init(this);` 来初始化 Toaster，完整代码如下：

``` cs
// 引用 Toaster Sdk 名称空间
using Com.Hjq.Toast;

[Application]
public class MainApplication : MauiApplication
{
    public MainApplication(IntPtr handle, JniHandleOwnership ownership)
        : base(handle, ownership)
    { }

    protected override MauiApp CreateMauiApp() => MauiProgram.CreateMauiApp();

    public override void OnCreate()
    {
        base.OnCreate();

        // 初始化 Toaster
        Toaster.Init(this);
    }
}
```

### 显示 Toast

创建一个空的 Maui 项目，并编辑 `MainPage.xaml.cs` 文件，在 `OnCounterClicked` 方法中编写调用 Toaster 的代码：

``` csharp
private void OnCounterClicked(object sender, EventArgs e)
{

    // 调用 Toaster 并显示字符串
#if ANDROID
    Com.Hjq.Toast.Toaster.Show("Toaster in Maui");
#endif

    count++;
    if (count == 1)
        CounterBtn.Text = $"Clicked {count} time";
    else
        CounterBtn.Text = $"Clicked {count} times";
    SemanticScreenReader.Announce(CounterBtn.Text);
}
```

效果如下：

{{< image src="./toaster.gif" caption="Toast in maui" class="img-h-600">}}

## 更多用法

可查看 [Toaster 文档](https://github.com/getActivity/Toaster#%E6%A1%86%E6%9E%B6-api-%E4%BB%8B%E7%BB%8D)来使用 Toaster 的更多功能。

## 源码获取

扫描下方二维码，关注公众号**捕获异常**，回复 **maui** 获取源码。
