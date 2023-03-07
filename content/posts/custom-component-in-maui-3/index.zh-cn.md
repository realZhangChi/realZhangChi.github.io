---
title: "在 Maui 中自绘组件2：事件与命令"
date: 2023-03-07T22:21:41+08:00
draft: false
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在 Maui 中基于 GraphicsView 视图，通过实现 IDrawable 来绘制自定义组件，并定义 Event 及 Command"
tags: ["maui"]
categories: ["maui"]
---

在这篇文章中，将为 `MagicButton` 定义 `Clicked` 事件及 `Command`，并调用事件和命令来增加计数值。

## 先决条件

- 阅读[在 Maui 中自绘组件1：绘制](https://zhangchi.io/posts/custom-component-in-maui-1/)
- 阅读[在 Maui 中自绘组件2：可绑定属性](https://zhangchi.io/posts/custom-component-in-maui-2/)

## 事件

## 定义事件

在 `GraphicsView` 中，定义了 `EndInteraction` 事件，当 `GraphicsView` 被按下并放开之后触发。要在 `MagicButton` 中实现点击，只需要在 `MagicButton` 中定义 `Clicked` 事件，并在 `EndInteraction` 事件处理程序中触发 `Clicked` 即可。

编辑 `MagicButton`，定义 `Clicked` 事件：

``` csharp
public event EventHandler Clicked;
```

定义 `OnEndInteraction` 方法来订阅 `EndInteraction` 事件，并在方法中触发 `Clicked` 事件：

``` csharp
public MagicButton() {
    Drawable = new MagicButtonDrawable();

    EndInteraction += OnEndInteraction;
}

private void OnEndInteraction(object sender, TouchEventArgs e) {
    Clicked?.Invoke(sender, e);
}
```

### 订阅事件

修改 `MainPage.xaml` 中的 `MagicButton`，设置 `Clicked` 事件处理程序：

``` xml
<components:MagicButton
    省略了其他属性设置......
    x:Name="MagicButton"
    Text="Click me"
    Clicked="MagicButton_OnClicked"></components:MagicButton>
```

修改 `MainPage.xaml.cs`，在 `MagicButton_OnClicked` 方法中处理点击事件：

``` csharp
private int _magicButtonCount = 0;
private void MagicButton_OnClicked(object sender, EventArgs e)
{
    _magicButtonCount++;

    MagicButton.Text = _magicButtonCount == 1 ?
        $"Clicked {_magicButtonCount} time" :
        $"Clicked {_magicButtonCount} times";
}
```

运行并点击 `MagicButton`，效果如下：

{{< image src="./event.gif" caption="Using Event" class="img-h-600">}}

## 命令

### 定义命令

在 `MagicButton` 中定义 `ICommand` 类型的可绑定属性，命名为 `Command`；定义 `object` 类型的命令参数可绑定属性，命名为 `CommandParameter`:

``` csharp
public static BindableProperty CommandProperty = BindableProperty.Create(
    nameof(Command),
    typeof(ICommand),
    typeof(MagicButton));

public ICommand Command {
    get => (ICommand)GetValue(CommandProperty);
    set => SetValue(CommandProperty, value);
}

public static BindableProperty CommandParameterProperty = BindableProperty.Create(
    nameof(CommandParameter),
    typeof(object),
    typeof(MagicButton));

public object CommandParameter {
    get => GetValue(CommandParameterProperty);
    set => SetValue(CommandParameterProperty, value);
}
```

修改 `OnEndInteraction` 方法，执行 `Command` 命令：

``` csharp
private void OnEndInteraction(object sender, TouchEventArgs e) {
    Clicked?.Invoke(sender, e);
    Command?.Execute(CommandParameter);
}
```

### 使用 ViewModel

在项目根目录下，创建 `ViewModels` 文件夹，并在其中创建 `MainViewModel`。在 `MainViewModel` 中定义 `ClickedCount` 属性用于 `MagicButton` 文本显示计数值，定义 `Command` 来增加 `ClickedCount` 计数值：

``` csharp
public class MainViewModel : INotifyPropertyChanged {

    private int _clickedCount;
    public int ClickedCount {
        set => SetField(ref _clickedCount, value, nameof(ClickedCount));
        get => _clickedCount;
    }

    public ICommand Command { get; set; }

    public MainViewModel() {
        Command = new Command(() => {
            ClickedCount++;
        });
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null) {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected bool SetField<T>(ref T field, T value, [CallerMemberName] string propertyName = null) {
        if (EqualityComparer<T>.Default.Equals(field, value)) return false;
        field = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}
```

更改 `MainPage.xaml.cs` 在构造函数中将 `BindingContext` 设为 `MainViewModel` 的实例：

``` csharp
public MainPage() {
    InitializeComponent();

    BindingContext = new MainViewModel();
}
```

更改 `MainPage.xaml` 中的 `MagicButton`，删除 `Clicked` 事件，并绑定 `Text` 及 `Command`；

``` xml
<components:MagicButton
    省略了其他属性设置......
    x:Name="MagicButton"
    x:DataType="viewModels:MainViewModel"
    Text="{Binding ClickedCount}"
    Command="{Binding Command}"></components:MagicButton>
```

运行并点击按钮，效果如下：

{{< image src="./command.gif" caption="Using Command" class="img-h-600">}}

## 推荐内容

- [Data binding and MVVM](https://learn.microsoft.com/en-us/dotnet/maui/xaml/fundamentals/mvvm?view=net-maui-7.0)
- [如何在Maui中使用依赖注入](https://zhangchi.io/posts/dependency-injection-in-maui/)
- [如何在Maui中使用Autofac](https://zhangchi.io/posts/use-autofac-in-maui/)
- [如何在 Maui 中全局处理异常](https://zhangchi.io/posts/handle-exception-in-maui/)