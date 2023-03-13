---
title: "在 Maui 中自绘组件3：点击动效"
date: 2023-03-13T09:51:16+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在 Maui 中基于 GraphicsView 视图，通过实现 IDrawable 来绘制自定义组件，设置点击动画效果"
tags: ["maui"]
categories: ["maui"]
---

在这篇文章中，将为 `MagicButton` 绘制点击动画效果，实现从点击位置开始，绘制一个不断扩大的半透明圆形。

## 先决条件

- 阅读[在 Maui 中自绘组件1：绘制](https://zhangchi.io/posts/custom-component-in-maui-1/)
- 阅读[在 Maui 中自绘组件2：可绑定属性](https://zhangchi.io/posts/custom-component-in-maui-2/)
- 阅读[在 Maui 中自绘组件3：事件与命令](https://zhangchi.io/posts/custom-component-in-maui-3/)

## 绘制点击动画

在 `MagicButtonDrawable` 中定义 `TouchPoint`、`AnimationPercent` 属性。`TouchPoint` 为点击焦点的位置坐标，`AnimationPercent` 为动画进度比例。

``` csharp
public PointF TouchPoint { get; set; }
public double AnimationPercent { get; set; }
```

定义 `DrawRippleEffect` 方法，绘制点击动画。

``` csharp
public void DrawRippleEffect(ICanvas canvas, RectF dirtyRect) {
    if (!dirtyRect.Contains(TouchPoint)) {
        return;
    }

    canvas.SaveState();

    var clippingPath = new PathF();
    clippingPath.AppendRoundedRectangle(
        dirtyRect.X,
        dirtyRect.Y,
        dirtyRect.Width,
        dirtyRect.Height,
        dirtyRect.Height / 2,
        dirtyRect.Height / 2,
        dirtyRect.Height / 2,
        dirtyRect.Height / 2
        );
    canvas.ClipPath(clippingPath);

    canvas.FillColor = Colors.White.WithAlpha(0.75f);
    canvas.Alpha = 0.25f;
    var minimumRippleEffectSize = 0.0f;
    var rippleEffectSize = minimumRippleEffectSize.Lerp(dirtyRect.Width, AnimationPercent);
    canvas.FillCircle(TouchPoint.X, TouchPoint.Y, rippleEffectSize);

    canvas.RestoreState();
}
```

通过 `ClipPath` 设置一个与当前 `dirtyRect` 位置大小相同的路径，绘制动画效果时将在此路径上绘制，否则动画效果将溢出 `MagicButton` 元素。然后根据 `AnimationPercent` 计算出要绘制的圆形动画效果的大小，并进行绘制。

## 触发绘制动画

在 `MagicButton` 中定义类型为 `IAnimationManager` 的字段，并在构造函数中进行实例化。

``` csharp
private readonly IAnimationManager _animationManager;

public MagicButton() {

    // ......

#if __ANDROID__
    _animationManager = new AnimationManager(new PlatformTicker(new Microsoft.Maui.Platform.EnergySaverListenerManager()));
#else
    _animationManager = new AnimationManager(new PlatformTicker());
#endif
}
```

定义 `AnimateRippleEffect` 来调用 `MagicButtonDrawable` 绘制动画。

``` csharp
public void AnimateRippleEffect() {
    if (Drawable is not MagicButtonDrawable drawable) {
        return;
    }

    var start = 0f;
    var end = 1f;

    _animationManager.Add(new Microsoft.Maui.Animations.Animation(
        (progress) => {
            drawable.AnimationPercent = start.Lerp(end, progress);
            Invalidate();
        },
        duration: 0.25,
        easing: Easing.SinInOut,
        finished: () => {
            drawable.AnimationPercent = 0;
        }));
}
```

在上述代码中通过 `Microsoft.Maui.Animations.Animation` 计算出了动画进度，并更新 `MagicButtonDrawable` 中的 `AnimationPercent` 值，然后调用 `Invalidate` 方法进行重绘。

定义 `OnStartInteraction` 方法来处理 `StartInteraction` 事件，将点击位置坐标赋值给 `MagicButtonDrawable`，并调用定义的 `AnimateRippleEffect` 方法，来实现点击时的动画效果。

``` csharp
public MagicButton() {
    // ......

    StartInteraction += OnStartInteraction;
}

private void OnStartInteraction(object sender, TouchEventArgs e) {
    if (Drawable is not MagicButtonDrawable drawable) {
        return;
    }

    drawable.TouchPoint = e.Touches[0];
    AnimateRippleEffect();
}
```

运行并点击按钮，效果如下：

{{< image src="./animation.gif" caption="BottomMenu in maui" class="img-h-600">}}

## 推荐内容

- [在 Maui 中使用 DialogX 展示输入对话框](https://zhangchi.io/posts/use-dialogx-in-maui-2/)
- [在 Maui Android 中使用 Toaster](https://zhangchi.io/posts/use-toaster-in-maui-android/)
- [如何在 Maui 中全局处理异常（异步方法）](https://zhangchi.io/posts/handle-exception-of-async-method-in-maui/)

## 源码获取

扫描下方二维码，关注公众号**捕获异常**，回复 **maui** 获取源码。
