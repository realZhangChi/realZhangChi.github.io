---
layout: post
title: "在 ASP.NET Core 中禁用HTTPS"
subtitle: 'Disable HTTPS in ASP.NET Core Application'
author: "Chi"
date: 2019-10-10 15:40
header-style: text
catalog: false
tags:
  - ASP.NET Core
---

我们在VS中新建一个 ASP.NET Core 项目的时候，可以选择是否配置使用HTTPS。如果选中了“为HTTPS配置”这个选项，在开发环境中想要临时禁用HTTPS，只使用HTTP应该怎样做呢？

![select https](/img/in-post/2019-10-10-net-core-disable-https/set-https.png)

在网上可能会找到很多通过在`Program.cs`文件中对`Kestrel`进行设置来禁用HTTPS的教程，或者是使用`UseUrls()`。如果我们只是需要在开发环境中禁用HTTPS，那么在设置`Kestrel`之前，首先检查两个位置，可能会让我们的工作更轻松。

## UseHttpsRedirection

首先查看一下`Startup.cs`的`Configure`方法中，是否配置了HTTPS重定向：

``` C#
app.UseHttpsRedirection();
```

`UseHttpsRedirection`为我们的应用添加了重定向HTTP请求到HTTPS请求的中间件。如果想要使用HTTP，那么就注释掉这行代码。

## Properties/launchSettings.json

在我们项目的`Properties/launchSettings.json`文件中，找到`applicationUrl`，类似如下代码：

``` json
...
"applicationUrl": "https://localhost:5001;http://localhost:5000",
...
```

将`https`地址删除，只保留`http`。

需要注意的是，通过 Visual Studio Code 或者 `dotnet new` 命令创建的项目，可能不会存在`launchSettings.json`文件，并且在发布项目的时候也不会将`launchSettings.json`文件包含进去，所以这种解决方案只对开发环境有效。

对于通过CLI来运行应用，也可以使用 `--urls` 参数来达到同样的效果。例如： `dotnet run --urls=http://0.0.0.0:5000,https://0.0.0.0:5001`。那么想要只使用HTTP，把命令中的`https`移除掉就好了。

## 参考

> [ASP.NET Core 2.1 + Kestrel (How to disable HTTPS)](https://stackoverflow.com/a/52533725)
