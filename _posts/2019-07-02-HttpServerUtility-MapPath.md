---
layout: post
title: "使用MapPath方法根据相对路径获取Web服务器上的物理文件路径"
subtitle: 'HttpServerUtility.MapPath(String) Method'
author: "Chi"
date: 2019-07-02 19:53
header-style: text
catalog: false
tags:
  - Web
---

今天在工作中遇到了上传文件的问题。对于上传文件，以前我利用FTP服务来实现过，而这次的需求是将文件保存在项目中，也就是Web应用程序中。

如何在Controller中接受文件，很容易从网上找到解决方案，可是在接收到文件后，如何对其指定路径并保存让我束手无策。因为对文件进行保存的时候，是要对其指定绝对路径的，但是在生产环境中，在不同的服务器上其绝对路径是不同的，相同的只有相对路径，所以我们就要通过代码获取到当前的路径才可以。

在网上搜到了一些方法，获取到的都是IIS Express的路径，和我想要的项目路径也就是Web应用程序的路径完全不是一个概念。

在这里感谢同事的热心帮助，学习到了利用`Server.MapPath(String)`方法来获取Web应用程序的根目录路径。

`MapPath(String)`是在`HttpServerUtility`类中的方法，`HttpServerUtility`类提供用于处理 Web 请求的 Helper 方法。命名空间是`System.Web`。

`MapPath`方法的作用是**返回与指定虚拟路径(相对路径)相对应的物理文件路径**。比如我们在项目根目录下存在一个`images`文件夹，那么这个文件夹在项目中的虚拟路径(相对路径)就是`/images`,我们要在运行时获取到`images`文件夹在Web服务器下的绝对路径，只需要调用`MapPath`方法并将`/images`(指定虚拟路径)作为参数传入即可。

``` C#
public string MapPath (string path);
```

### 参数

`path` String

Web 应用程序中的虚拟路径。

### 返回值

String

对应于 `path` 的 Web 服务器上的物理文件路径。

### 异常

[HttpException](https://docs.microsoft.com/zh-cn/dotnet/api/system.web.httpexception?view=netframework-4.8)
当前[HttpContext](https://docs.microsoft.com/zh-cn/dotnet/api/system.web.httpcontext?view=netframework-4.8)为`null`，

或`path`是物理路径但应为虚拟路径。

## 示例

下面的示例演示如何检索的物理文件的相对虚拟路径。 代码驻留在 web 页的代码隐藏文件，并利用默认`Server`对象。

``` C#
public partial class _Default : Page
{
    protected void Page_Load(object sender, EventArgs e)
    {
        string pathToFiles = Server.MapPath("/UploadedFiles");
    }
}
```

下一个示例非常相似与前面的示例，但它演示如何检索中的类中不是代码隐藏文件中的物理路径。

``` C#
public class SampleClass
{
    public string GetFilePath()
    {
        return HttpContext.Current.Server.MapPath("/UploadedFiles");
    }
}
```

## 注解

如果`path`是`null`，则`MapPath`方法返回包含当前请求路径的目录的完整物理路径。使用`MapPath`方法所需要的参数（相对路径）无需是已经存在的文件或文件夹，但是需要注意的是，我们不可以指定此 Web 应用程序之外的路径。

## 参考

[HttpServerUtility Class](https://docs.microsoft.com/zh-cn/dotnet/api/system.web.httpserverutility?view=netframework-4.8)

[HttpServerUtility.MapPath(String) Method](https://docs.microsoft.com/zh-cn/dotnet/api/system.web.httpserverutility.mappath?view=netframework-4.8)
