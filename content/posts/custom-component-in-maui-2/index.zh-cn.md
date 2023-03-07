---
title: "在 Maui 中自绘组件2：可绑定属性"
date: 2023-03-06T20:06:58+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在 Maui 中基于 GraphicsView 视图，通过实现 IDrawable 来绘制自定义组件，并为自定义组件设置可绑定属性"
tags: ["maui"]
categories: ["maui"]
---

在这篇文章中，将为 `MagicButton` 添加可绑定属性，并根据可绑定属性进行绘制。

## 先决条件

- 阅读[在 Maui 中自绘组件1：绘制](https://zhangchi.io/posts/custom-component-in-maui-1/)

## 更新 `MagicButtonDrawable`

在 `MagicButtonDrawable` 中，为绘制时所需的颜色、字体大小等创建属性。

``` csharp
public Color StrokeColor { get; set; }
public float StrokeThickness { get; set; }
public Color BackgroundColor { get; set; }
public int FontSize { get; set; }
public Color FontColor { get; set; }
public string Text { get; set; }
```

更新 `DrawBackground` 等方法，依据上述属性值进行绘制。

``` csharp
// 以 DrawBackground 为例，其他方法同理

public void DrawBackground(ICanvas canvas, RectF dirtyRect) {

    canvas.SaveState();

    // 使用 BackgroundColor 属性
    canvas.SetFillPaint(new SolidPaint(BackgroundColor), dirtyRect);

    // 使用 StrokeThickness 属性
    var x = dirtyRect.X + StrokeThickness;
    var y = dirtyRect.Y + StrokeThickness;
    var width = dirtyRect.Width - StrokeThickness;
    var height = dirtyRect.Height - StrokeThickness;

    canvas.FillRoundedRectangle(x, y, width, height, height / 2);

    canvas.RestoreState();
}
```

## 定义可绑定属性

为 `MagicButton` 定义 `StrokeColor`、`StrokeThickness`、`FontSize`、`FontColor`、`Text`等可绑定属性。在 `GraphicsView` 中已定义 `BackgroundColor` 可绑定属性，无需重复定义。

``` csharp
// 以 StrokeColor 可绑定属性为例，其他方法同理

public static BindableProperty StrokeColorProperty = BindableProperty.Create(
    nameof(StrokeColor),
    typeof(Color),
    typeof(MagicButton),
    null,
    propertyChanged: (bindable, value, newValue) => {
        if (bindable is MagicButton magicButton) {
            // TODO：处理属性值变更，进行重绘
        }
    });

public Color StrokeColor {
    get => (Color)GetValue(FontColorProperty);
    set => SetValue(FontColorProperty, value);
}
```

定义 `UpdateStrokeColor` 等方法，在属性值变更后，更新 `MagicButtonDrawable` 中相应属性值，并进行重绘。

``` csharp
// 以 UpdateStrokeColor 为例，其他同理

public void UpdateStrokeColor() {
    if (Drawable is not MagicButtonDrawable drawable) {
        return;
    }
    if (StrokeColor is null) {
        return;
    }

    // 更新 MagicButtonDrawable 中 StrokeColor 值
    drawable.StrokeColor = StrokeColor;

    // 通知并进行重绘
    Invalidate();
}

```

更新 `StrokeColorProperty` 等属性中的 `propertyChanged`，调用相应方法以在属性值变更时更新 `MagicButtonDrawable` 并重绘。

``` csharp
public static BindableProperty StrokeColorProperty = BindableProperty.Create(
    nameof(StrokeColor),
    typeof(Color),
    typeof(MagicButton),
    null,
    propertyChanged: (bindable, value, newValue) => {
        if (bindable is MagicButton magicButton) {
            magicButton.UpdateStrokeColor();
        }
    });
```

## 设置 MagicButton 属性值

更新 `MainPage.xaml`，修改 `MagicButton` 的使用，设置属性，并将 `Text` 属性绑定到 `CounterBtn` 中的 `Text`，来查看绑定效果。

``` csharp
<!--使用自定义的 MagicButton-->
<components:MagicButton
    HeightRequest="60"
    WidthRequest="200"
    BackgroundColor="Blue"
    StrokeColor="{StaticResource Black}"
    StrokeThickness="3"
    FontColor="{StaticResource Cyan300Accent}"
    FontSize="25"
    BindingContext="{x:Reference CounterBtn}"
    Text="{Binding Text}"></components:MagicButton>

<Button
    x:Name="CounterBtn"
    Text="Click me"
    SemanticProperties.Hint="Counts the number of times you click"
    Clicked="OnCounterClicked"
    HorizontalOptions="Center" />
```

运行并点击 `CounterBtn`， 效果如下：

{{< image src="./custom-button.gif" caption="Custom Button" class="img-h-600">}}

## 推荐内容

- [在 Maui 中自绘组件1：绘制](https://zhangchi.io/posts/custom-component-in-maui-1/)
- [如何在Maui中使用Autofac](https://zhangchi.io/posts/use-autofac-in-maui/)
- [在 Maui 中使用 DialogX 展示消息对话框](https://zhangchi.io/posts/use-dialogx-in-maui/)
