---
title: "如何在 Maui 中使用 EasyFloat 显示浮窗"
date: 2023-03-01T09:45:29+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在 Maui 项目中引入 EasyFloat 原生安卓的绑定类库，来显示自定义浮窗"
tags: ["maui", "binding library"]
categories: ["maui"]
---

## EasyFloat Android Sdk

EasyFloat 是一个开源的 Android 浮窗框架，支持可拖拽悬浮窗口，支持页面过滤、自定义动画，可设置单页面浮窗、前台浮窗、全局浮窗，浮窗权限按需自动申请等多种功能 。
EasyFloat 项目在 github 上开源，地址：[https://github.com/princekin-f/EasyFloat](https://github.com/princekin-f/EasyFloat).

## EasyFloat 绑定库

在 [MauiBinding](https://github.com/realZhangChi/MauiBinding) 项目中已为 EasyFloat 创建好了绑定类库，并可在 [Nuget](https://www.nuget.org/packages/Chi.MauiBinding.EasyFloat.Android) 上下载。通过 Nuget 包管理器或 CLI 安装 `Chi.MauiBinding.EasyFloat.Android` 即可使用 EasyFloat。

## 如何使用

根据 [EasyFloat 文档](https://github.com/princekin-f/EasyFloat#%E4%B8%80%E8%A1%8C%E4%BB%A3%E7%A0%81%E6%90%9E%E5%AE%9Aandroid%E6%B5%AE%E7%AA%97%E6%B5%AE%E7%AA%97%E4%BB%8E%E6%9C%AA%E5%A6%82%E6%AD%A4%E7%AE%80%E5%8D%95)描述，定义了以下方法用于显示浮窗：

``` csharp
Com.Lzf.Easyfloat.EasyFloat
    .With(your_android_app_activity)
    .SetLayout(your_float_view)
    .Show();
```

其中 `With` 方法接收类型为 `Android.Content.Context` 的安卓平台 `Activity`，可通过安装 Nuget 包 `Microsoft.Maui.Essentials` 并调用 `Microsoft.Maui.ApplicationModel.Platform.CurrentActivity` 获取当前 Activity。

`SetLayout` 方法接收安卓平台的 `Android.Views.View`，此参数即为要显示的浮窗。在 Maui 中，可以创建 Maui 的 `ContentView`，并调用其扩展方法 `ToPlatform` 将 Maui 的 `ContentView` 转换为安卓平台的 `Android.Views.View`。

## 项目实践

### 初始化项目

创建一个新的 Maui 项目，并添加对Nuget包 `Chi.MauiBinding.EasyFloat.Android`、`Microsoft.Maui.Essentials` 引用。

### 创建浮窗

在项目中新建一个 `ContentView` 命名为 `FloatView`，修改 XAML 代码调整样式：

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentView xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="YOUR_NAMESPACE_TO_FLOAT_VIEW_CLASS.FloatView">

    <Border
        HeightRequest="50"
        WidthRequest="200"
        BackgroundColor="{StaticResource Cyan300Accent}">
        <Border.StrokeShape>
            <RoundRectangle CornerRadius="25"
                            StrokeThickness="0"/>
        </Border.StrokeShape>

        <VerticalStackLayout
            VerticalOptions="Center">
            <Label 
            Text="Welcome to .NET MAUI!"
            VerticalOptions="Center" 
            HorizontalOptions="Center" />
        </VerticalStackLayout>
    </Border>
</ContentView>
```

{{< admonition tip "替换代码" >}}
替换上述代码中的 `YOUR_NAMESPACE_TO_FLOAT_VIEW_CLASS` 为你创建的 `FloatView` 所在名称空间。
{{< /admonition >}}

### 显示浮窗

更改 `MainPage.xaml.cs` 中的代码，加入显示浮窗的代码：

``` csharp
private bool _isFloatViewShow = false;
private const string FloatViewTag = "FloatView";
private void OnCounterClicked(object sender, EventArgs e) {
#if ANDROID
    if (!_isFloatViewShow &&
        Microsoft.Maui.ApplicationModel.Platform.CurrentActivity != null &&
        Application.Current?.Handler?.MauiContext != null) {
        EasyFloat
            .With(Microsoft.Maui.ApplicationModel.Platform.CurrentActivity)
            .SetTag(FloatViewTag)
            .SetLayout(new FloatView().ToPlatform(Application.Current.Handler.MauiContext))
            .SetLocation(250, 1000)
            .SetDragEnable(true)
            .Show();
        _isFloatViewShow = true;
    }
    else {
        EasyFloat.Dismiss(FloatViewTag);
        _isFloatViewShow = false;
    }
#endif
}
```

运行项目并点击按钮，即可弹出浮窗。

{{< image src="./easyfloat.gif" caption="EasyFloat in maui" class="img-h-600">}}

## 推荐内容

关于 EasyFloat 的更多用法，请参考 [EasyFloat 文档](https://github.com/princekin-f/EasyFloat#easyfloatandroid%E6%82%AC%E6%B5%AE%E7%AA%97%E6%A1%86%E6%9E%B6)。

## 源码获取

扫描下方二维码，关注公众号**捕获异常**，回复 **maui** 获取源码。
