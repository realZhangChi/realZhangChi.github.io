---
title: "在 Maui 中自绘组件5：状态与视觉效果"
date: 2023-03-14T08:59:53+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在 Maui 中基于 GraphicsView 视图，通过实现 IDrawable 来绘制自定义组件，设置不同状态视觉效果"
tags: ["maui"]
categories: ["maui"]
---

在这篇文章中，将为 `MagicButton` 设置不同状态的视觉效果。

## 先决条件

- 阅读[在 Maui 中自绘组件1：绘制](https://zhangchi.io/posts/custom-component-in-maui-1/)
- 阅读[在 Maui 中自绘组件2：可绑定属性](https://zhangchi.io/posts/custom-component-in-maui-2/)
- 阅读[在 Maui 中自绘组件3：事件与命令](https://zhangchi.io/posts/custom-component-in-maui-3/)
- 阅读[在 Maui 中自绘组件4：点击动效](https://zhangchi.io/posts/custom-component-in-maui-4/)

## 资源字典

在 `Resources/Styles` 中新建资源字典文件，并定义正常、点击等不同状态的视觉效果样式。

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:components="clr-namespace:CustomButton.Components"
             x:Class="CustomButton.Resources.Styles.MagicButtonStyles">

    <Style TargetType="components:MagicButton">
        <Setter Property="VisualStateManager.VisualStateGroups">
            <VisualStateGroupList>
                <VisualStateGroup x:Name="CommonStates">
                    <VisualState x:Name="Normal">
                        <VisualState.Setters>
                            <Setter Property="BackgroundColor" Value="Blue"/>
                        </VisualState.Setters>
                    </VisualState>
                    <VisualState x:Name="Clicked">
                        <VisualState.Setters>
                            <Setter Property="BackgroundColor" Value="DarkBlue"></Setter>
                        </VisualState.Setters>
                    </VisualState>
                </VisualStateGroup>
            </VisualStateGroupList>
        </Setter>
    </Style>
</ResourceDictionary>
```

将 `MagicButtonStyles` 添加到 `Resources/Styles/Styles.xaml` 中。

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<?xaml-comp compile="true" ?>
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="MagicButtonStyles.xaml"></ResourceDictionary>
    </ResourceDictionary.MergedDictionaries>

    <!--......-->

</ResourceDictionary>
```

## 状态管理

更改 `MagicButton` 中的代码，在 `OnStartInteraction` 和 `OnEndInteraction` 方法中，通过 `VisualStateManager` 在按钮被按下时，将状态设为 `Clicked`，在按钮被松开时，将状态设为 `Normal`。

``` csharp
private void OnStartInteraction(object sender, TouchEventArgs e)
{
    VisualStateManager.GoToState(this, "Clicked");
}

private void OnEndInteraction(object sender, TouchEventArgs e)
{
    VisualStateManager.GoToState(this, "Normal");
}
```

## 效果展示

运行项目，点击按钮并松开，可以看到按钮会根据不同状态展示不同样式。

{{< image src="./visual-state.gif" caption="Visual state" class="img-h-600">}}

## 推荐内容

- [如何在Maui中使用依赖注入](https://zhangchi.io/posts/dependency-injection-in-maui/)
- [如何在Maui中使用Autofac](https://zhangchi.io/posts/use-autofac-in-maui/)
- [在 Maui Android 中使用 Toaster](https://zhangchi.io/posts/use-toaster-in-maui-android/)

## 源码获取

扫描下方二维码，关注公众号**捕获异常**，回复 **maui** 获取源码。
