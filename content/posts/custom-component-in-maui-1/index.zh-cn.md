---
title: "在 Maui 中自绘组件1：绘制"
date: 2023-03-06T10:49:04+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在 Maui 中基于 GraphicsView 视图，通过实现 IDrawable 来自定义绘制组件"
tags: ["maui"]
categories: ["maui"]
---

在这篇文章中，将自定义一个简单的按钮组件，绘制边框、背景、文字元素。

## GraphicsView

Maui 中提供了 `GraphicsView` 视图，可通过继承 `GraphicsView` 视图来自定义组件。

`GraphicsView` 中定义了类型为 `IDrawable` 的属性，在渲染时，将调用 `IDrawable` 中的 `Draw` 方法来绘制组件。

## 创建 MagicButtonDrawable

新建一个空的 `Maui` 项目，在项目根目录下创建 `Components` 文件夹，在其中创建 `MagicButtonDrawable` 类，并继承 `IDrawable`。`MagicButtonDrawable` 将负责自定义组件的绘制。

``` csharp
public class MagicButtonDrawable : IDrawable
{
    public void Draw(ICanvas canvas, RectF dirtyRect)
    {
    }
}
```

### 绘制边框

定义方法 `DrawStroke` 来绘制边框：

``` csharp
public void DrawStroke(ICanvas canvas, RectF dirtyRect)
{
    canvas.SaveState();

    canvas.SetFillPaint(new SolidPaint(Brush.LightBlue.Color), dirtyRect);

    canvas.FillRoundedRectangle(dirtyRect.X, dirtyRect.Y, dirtyRect.Width, dirtyRect.Height, dirtyRect.Height / 2);

    canvas.RestoreState();
}
```

在此方法中调用了 `ICanvas` 的 `FillRoundedRectangle` 方法，绘制了一个填充色为 LightBlue 的圆角矩形。下一步将在此矩形之上，再绘制一个宽高小于此图案的不同填充色的圆角矩形，来实现边框的效果。

### 绘制背景

定义方法 `DrawBackground` 来绘制背景：

``` csharp
public void DrawBackground(ICanvas canvas, RectF dirtyRect)
{
    canvas.SaveState();

    canvas.SetFillPaint(new SolidPaint(Brush.Blue.Color), dirtyRect);

    var strokeThickness = 3;
    var x = dirtyRect.X + strokeThickness;
    var y = dirtyRect.Y + strokeThickness;
    var width = dirtyRect.Width - strokeThickness * 2;
    var height = dirtyRect.Height - strokeThickness * 2;

    canvas.FillRoundedRectangle(x, y, width, height, height / 2);

    canvas.RestoreState();
}
```

将边框厚度设为3，那么将绘制起始点的 X、Y坐标都加上边框的宽度， 并将宽度和高度都减去两个边框的厚度，来进行绘制，即可得到底层一个大的圆角矩形，其上一个略小的圆角矩形，从而实现边框的效果。

{{< admonition tip >}}
`dirtyRect` 的 `X` 和 `Y` 为绘制区域的左上角坐标，在 `canvas` 上进行绘制将根据 `dirtyRect` 的 `X` 和 `Y` 从左上角开始绘制。
{{< /admonition >}}

### 绘制文本

定义方法 `DrawText` 来绘制按钮中的文本内容：

``` csharp
public void DrawText(ICanvas canvas, RectF dirtyRect)
{
    canvas.SaveState();

    canvas.FontColor = Brush.White.Color;
    canvas.FontSize = 16;
    canvas.DrawString("Magic Button", dirtyRect.X, dirtyRect.Y, dirtyRect.Width, dirtyRect.Height,
        HorizontalAlignment.Center,
        VerticalAlignment.Center);

    canvas.RestoreState();
}
```

## 创建 MagicButton

在 `Components` 文件夹中创建 `MagicButton` 类，并继承 `GraphicsView`。通过构造函数将 `Drawable` 属性设置为 `MagicButtonDrawable` 的实例。

``` csharp
public class MagicButton : GraphicsView
{
    public MagicButton()
    {
        Drawable = new MagicButtonDrawable();
    }
}
```

## 使用 MagicButton

修改 `MainPage.xaml` 引用 `MagicButton` 名称空间，并添加 `MagicButton` 组件：

``` xml

<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:components="clr-namespace:YOUR_MAGICBUTTON_NAMESPACE"
             x:Class="YOUR_ROOT_NAMESAPCE.MainPage">

    <ScrollView>
        <VerticalStackLayout
            Spacing="25"
            Padding="30,0"
            VerticalOptions="Center">

            <!-- ...... -->

            <!--使用自定义的 MagicButton-->
            <components:MagicButton
                HeightRequest="50"
                WidthRequest="150"></components:MagicButton>

            <Button
                x:Name="CounterBtn"
                Text="Click me"
                SemanticProperties.Hint="Counts the number of times you click"
                Clicked="OnCounterClicked"
                HorizontalOptions="Center" />

        </VerticalStackLayout>
    </ScrollView>

</ContentPage>

```

效果如下：

{{< image src="./custom_button.png" caption="Custom Button" class="img-h-600">}}
