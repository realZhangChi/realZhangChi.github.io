---
title: "在 Maui 中使用 DialogX 展示底部菜单"
date: 2023-03-02T22:18:33+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在 Maui 项目中引入 DialogX 原生安卓的绑定类库，显示对话框、菜单、提示效果、输入框"
tags: ["maui", "binding library"]
categories: ["maui"]
---

## DialogX Android Sdk

DialogX 是一个开源的 Android 对话框组件库，方便易用，扩展性强，轻松实现各种对话框、菜单和提示效果。
DialogX 项目在 github 上开源，地址：[https://github.com/kongzue/DialogX](https://github.com/kongzue/DialogX)。

## DialogX 绑定库

在 [MauiBinding](https://github.com/realZhangChi/MauiBinding) 项目中已为 DialogX 创建好了绑定类库，并可在 [Nuget](https://www.nuget.org/packages/Chi.MauiBinding.Android.DialogX) 下载。通过 Nuget 包管理器或 CLI 安装 `Chi.MauiBinding.Android.DialogX` 即可使用 DialogX。

## 初始化

创建一个新的 Maui 项目，并添加Nuget包 `Chi.MauiBinding.Android.DialogX` 引用。

打开并编辑 `DialogXMaui/Platforms/Android/MainApplication.cs`, 重写 `OnCreate` 方法，对 DialogX 进行初始化。

``` csharp
public override void OnCreate() {
    base.OnCreate();

    Com.Kongzue.Dialogx.DialogX.Init(this);
}
```

## 展示底部菜单

通过查阅 [DialogX Wiki](https://github.com/kongzue/DialogX/wiki/%E5%BA%95%E9%83%A8%E5%AF%B9%E8%AF%9D%E6%A1%86-BottomDialog-%E5%92%8C%E5%BA%95%E9%83%A8%E8%8F%9C%E5%8D%95-BottomMenu) 可知，调用 `BottomDialog.Show()` 可弹出底部菜单。

打开并编辑 `MainPage.xaml.cs`，修改 `OnCounterClicked` 方法如下：

``` csharp
private void OnCounterClicked(object sender, EventArgs e)
{
    BottomMenu.Show("菜单", new[] { "菜单1", "菜单2", "菜单3" });
}
```

运行项目并点击按钮，将弹出底部菜单。

{{< image src="./buttom-menu.gif" caption="BottomMenu in maui" class="img-h-600">}}

### 菜单项选中回调

根据 [DialogX Wiki](https://github.com/kongzue/DialogX/wiki/%E5%BA%95%E9%83%A8%E5%AF%B9%E8%AF%9D%E6%A1%86-BottomDialog-%E5%92%8C%E5%BA%95%E9%83%A8%E8%8F%9C%E5%8D%95-BottomMenu#%E6%98%BE%E7%A4%BA%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E8%8F%9C%E5%8D%95) 所描述，可通过 `SetOnMenuItemClickListener()` 方法来设置菜单项点击的回调处理。

在 Visual Studio 中转到 `SetOnMenuItemClickListener` 方法的定义，可以发现 `SetOnMenuItemClickListener` 方法接受类型为 `IOnMenuItemClickListener` 的参数。在 C# 中不支持匿名类的创建，因此需要先创建一个继承 `IOnMenuItemClickListener` 接口的类，在调用 `SetOnMenuItemClickListener` 的时候实例化并传入。

创建名为 `MenuItemClickListener` 的类：

``` csharp
public class MenuItemClickListener : Java.Lang.Object, IOnMenuItemClickListener
{
    public Func<BottomMenu, string, int, bool> OnMenuItemClick;

    public bool OnClick(Object p0, ICharSequence p1, int p2) {
        if (p0 is BottomMenu bm) {
            return OnMenuItemClick?.Invoke(bm, p1.ToString(), p2) ?? false;
        }

        return false;
    }
}
```

`OnClick` 是定义在 `IOnMenuItemClickListener` 接口中的方法，在点击某个菜单项时被调用。当 `OnClick` 被调用时，将执行 `OnMenuItemClick`。`OnMenuItemClick` 在实例化 `MenuItemClickListener` 时赋值。

修改 `OnCounterClicked` 方法，设置菜单项点击回调，并在回调方法中调用 DialogX 的 PopTip 弹出文本提示，展示选中菜单名称及其索引：

``` csharp
private void OnCounterClicked(object sender, EventArgs e) {

    BottomMenu.Show("菜单", new[] { "菜单1", "菜单2", "菜单3" })?
        .SetOnMenuItemClickListener(new MenuItemClickListener() {
            OnMenuItemClick = (menu, menuName, menuIndex) => {
                PopTip.Show($"{menuName} has been clicked! index: {menuIndex}");
                return false;
            }
        });
}
```

{{< admonition tip >}}
回调函数返回`false`时，底部菜单关闭；回调函数返回`true`时，底部菜单将不会关闭。
{{< /admonition >}}

运行项目并点击按钮，点击底部菜单中的某一菜单项后，将关闭底部菜单并弹出 PopTip 提示，展示选中菜单名称及其索引。

{{< image src="./buttom-menu-callback.gif" caption="BottomMenu in maui" class="img-h-600">}}

## 推荐内容

请查阅 [DialogX Wiki](https://github.com/kongzue/DialogX/wiki/%E5%BA%95%E9%83%A8%E5%AF%B9%E8%AF%9D%E6%A1%86-BottomDialog-%E5%92%8C%E5%BA%95%E9%83%A8%E8%8F%9C%E5%8D%95-BottomMenu) 获取主题设置、自定义布局、自定义动画等更多用法。
