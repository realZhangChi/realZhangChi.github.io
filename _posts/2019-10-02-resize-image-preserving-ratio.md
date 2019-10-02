---
layout: post
title: "如何在保持原有宽高比的条件下更改图片尺寸"
subtitle: 'resize image while preserving aspect ratio'
author: "Chi"
date: 2019-10-02 18:10
header-style: text
catalog: false
tags:
  - Image
---

在改变图片尺寸的时候，如何保持原有图片的宽高比并将多余部分填充颜色？

## 解决方案

先创建一个目标尺寸的Graphics绘图图层，然后将原始图片进行等比缩放，再将缩放后的图片放置到绘图图层中间，并另存为新图片即可。

首先加载原始图片，并获取原始宽高值：

``` C#
// tempFullFileName为原始图片路径
var image = Image.FromFile(tempFullFileName);

var originalWidth = image.Width;
var originalHeight = image.Height;
```

新建一个矢量图类，将其加载到`Graphics`绘图图层类中,并设置初始化参数：

``` C#
// 比如目标尺寸是1080*1080
var thumbnail = new Bitmap(1080, 1080);
var graphic = Graphics.FromImage(thumbnail);
graphic.InterpolationMode = InterpolationMode.HighQualityBicubic;
graphic.PixelOffsetMode = PixelOffsetMode.HighQuality;
graphic.CompositingQuality = CompositingQuality.HighQuality;
```

计算出原始图片转换到目标尺寸图片的缩放比例：

``` C#
var ratioX = (double)1080 / (double)originalWidth;
var ratioY = (double)1080 / (double)originalHeight;
// 上述两个值中最小值则是缩放比例
var ratio = ratioX < ratioY ? ratioX : ratioY;
``` 

计算出原始图片缩放后的宽度和高度：

``` C#
var newHeight = Convert.ToInt32(originalHeight * ratio);
var newWidth = Convert.ToInt32(originalWidth * ratio);
```

计算出缩放后的图片相对Graphics左上角的坐标， 横坐标和纵坐标必然有一个值是为0的：

``` C#
var posX = Convert.ToInt32((1080 - (originalWidth * ratio)) / 2);
var posY = Convert.ToInt32((1080 - (originalHeight * ratio)) / 2);
```

将Graphics绘图图层用白色填充，然后将缩放后的图片，根据计算出来的宽高、位置绘制到Graphics绘图图层中：

``` C#
graphic.Clear(Color.White); // 填充为白色
graphic.DrawImage(image, posX, posY, newWidth, newHeight);
```

最后将通过Graphics绘制之后的矢量图保存下来即可：

``` C#
thumbnail.Save(fullFileName);
```

## 源码

``` C#
private static void ResizeImage(string tempFullFileName, string fullFileName)
{
    using (var image = Image.FromFile(tempFullFileName))
    {
        var originalWidth = image.Width;
        var originalHeight = image.Height;

        var thumbnail = new Bitmap(1080, 1080);
        var graphic = Graphics.FromImage(thumbnail);
        graphic.InterpolationMode = InterpolationMode.HighQualityBicubic;
        graphic.SmoothingMode = SmoothingMode.HighQuality;
        graphic.PixelOffsetMode = PixelOffsetMode.HighQuality;
        graphic.CompositingQuality = CompositingQuality.HighQuality;

        var ratioX = (double)1080 / (double)originalWidth;
        var ratioY = (double)1080 / (double)originalHeight;
        var ratio = ratioX < ratioY ? ratioX : ratioY;

        var newHeight = Convert.ToInt32(originalHeight * ratio);
        var newWidth = Convert.ToInt32(originalWidth * ratio);

        var posX = Convert.ToInt32((1080 - (originalWidth * ratio)) / 2);
        var posY = Convert.ToInt32((1080 - (originalHeight * ratio)) / 2);

        graphic.Clear(Color.White); // white padding
        graphic.DrawImage(image, posX, posY, newWidth, newHeight);

        thumbnail.Save(fullFileName);
    }
}
```

> [c# Image resizing to different size while preserving aspect ratio](https://stackoverflow.com/a/2001692)