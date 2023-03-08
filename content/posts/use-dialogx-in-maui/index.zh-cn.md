---
title: "在 Maui 中使用 DialogX 展示消息对话框"
date: 2023-03-02T21:24:18+08:00
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

## 展示消息对话框

通过查阅 [DialogX Wiki](https://github.com/kongzue/DialogX/wiki/%E5%9F%BA%E7%A1%80%E5%AF%B9%E8%AF%9D%E6%A1%86-MessageDialog-%E5%92%8C-%E8%BE%93%E5%85%A5%E5%AF%B9%E8%AF%9D%E6%A1%86-InputDialog) 可知，调用 `MessageDialog.Show()` 可弹出消息对话框。

打开并编辑 `MainPage.xaml.cs`，修改 `OnCounterClicked` 方法如下：

``` csharp
private void OnCounterClicked(object sender, EventArgs e)
{
    MessageDialog.Show("Hi", "Maui", "确定");
}
```

运行项目并点击按钮，将弹出消息对话框。

{{< image src="./message-dialog.gif" caption="MessageDialog in maui" class="img-h-600">}}

### 确认回调

根据 [DialogX Wiki](https://github.com/kongzue/DialogX/wiki/%E5%9F%BA%E7%A1%80%E5%AF%B9%E8%AF%9D%E6%A1%86-MessageDialog-%E5%92%8C-%E8%BE%93%E5%85%A5%E5%AF%B9%E8%AF%9D%E6%A1%86-InputDialog#%E6%8C%89%E9%92%AE%E7%82%B9%E5%87%BB%E5%9B%9E%E8%B0%83) 所描述，可通过 `MessageDialog.Show().SetOkButton()` 方法来设置确认按钮的回调处理。

在 Visual Studio 中转到 `SetOkButton` 方法的定义，可以发现 `SetOkButton` 方法接受类型为 `IOnDialogButtonClickListener` 的参数。在 C# 中不支持匿名类的创建，因此需要先创建一个继承 `IOnDialogButtonClickListener` 接口的类，在调用 `SetOkButton` 的时候实例化并传入。

创建名为 `OkClickListener` 的类：

``` csharp
public class OkClickListener : Java.Lang.Object, IOnDialogButtonClickListener {
    public Func<MessageDialog, View, bool> OnOkButtonClicked;

    public bool OnClick(Object p0, View p1) {
        if (p0 is MessageDialog md) {
            return OnOkButtonClicked?.Invoke(md, p1) ?? false;
        }

        return false;
    }
}
```

`OnClick` 是定义在 `IOnDialogButtonClickListener` 接口中的方法，在点击确认按钮时被调用。当 `OnClick` 被调用时，将执行 `OnOkButtonClicked`。`OnOkButtonClicked` 在实例化 `OkClickListener` 时赋值。

修改 `OnCounterClicked` 方法，设置按钮回调，并在回调方法中调用 DialogX 的 PopTip 弹出文本提示：

``` csharp
private void OnCounterClicked(object sender, EventArgs e)
{
    MessageDialog.Show("Hi", "Maui", "确定")
        .SetOkButton(new OkClickListener() {
            OnOkButtonClicked = (dialog, view) => {
                PopTip.Show("Hello! Ok Button has been clicked!");
                return false;
            }
        });
}
```

{{< admonition tip >}}
回调函数返回`false`时，消息对话框关闭；回调函数返回`true`时，消息对话框将不会关闭。
{{< /admonition >}}

运行项目并点击按钮，点击消息对话框的“确定”按钮后，将关闭消息对话框并弹出 PopTip 提示。

{{< image src="./message-dialog-callback.gif" caption="MessageDialog in maui" class="img-h-600">}}

## 推荐内容

- [在 Maui 中使用 DialogX 展示输入对话框](https://zhangchi.io/posts/use-dialogx-in-maui-2/)
- [在 Maui 中使用 DialogX 展示底部菜单](https://zhangchi.io/posts/use-dialogx-in-maui-3/)
- [在 Maui 中自绘组件1：绘制](https://zhangchi.io/posts/custom-component-in-maui-1/)

请查阅 [DialogX Wiki](https://github.com/kongzue/DialogX/wiki/%E5%9F%BA%E7%A1%80%E5%AF%B9%E8%AF%9D%E6%A1%86-MessageDialog-%E5%92%8C-%E8%BE%93%E5%85%A5%E5%AF%B9%E8%AF%9D%E6%A1%86-InputDialog) 获取主题设置、自定义布局、自定义动画等更多用法。

## 源码获取

扫描下方二维码，关注公众号**捕获异常**，回复 **maui** 获取源码。
