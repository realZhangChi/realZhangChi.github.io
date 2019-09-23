---
layout: post
title: "移除 ASP.NET 项目返回的 X-AspNetMvc-Version 等 Response Header"
subtitle: 'Remove X-AspNetMvc-Version etc reponse header in ASP.NET project'
author: "Chi"
date: 2019-09-23 16:25
header-style: text
catalog: false
tags:
  - ASP.NET MVC
---

移除 ASP.NET MVC 项目中，HTTP 请求的Response Header中的 X-AspNetMvc-Version， Server， X-AspNet-Version， X-Powered-By

![Reponse Header](/img/in-post/2019-09-23-remove-aspnet-version/2019-09-23-reponse-header.png)

## X-AspNetMvc-Version

将 `MvcHandler.DisableMvcResponseHeader = true;` 添加到 `Global.asax.cs` 文件的 `Application_Start` 方法里即可。

``` C#
public class MvcApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
            // 隐藏Response Header中的X-AspNetMvc-Version
            MvcHandler.DisableMvcResponseHeader = true;
    }
}
```

## Server

在 `Global.asax.cs` 文件的 `Application_Start` 方法中添加方法 `Application_PreSendRequestHeaders`：

``` C#
public class MvcApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        some code...
    }

    /// <summary>
    /// 隐藏 Response Header 中的Server节点（IIS版本信息）
    /// </summary>
    /// <param name="sender"></param>
    /// <param name="e"></param>
    protected void Application_PreSendRequestHeaders(object sender, EventArgs e)
    {
        if (sender is HttpApplication application)
        {
            application.Context.Response.Headers.Remove("Server");
        }
    }
}
```

## X-AspNet-Version

在 `web.config` 文件中将 `enableVersionHeader` 值改为 `false`

``` XML
<configuration>
    <system.web>
        <httpRuntime enableVersionHeader="false" />
    </system.web>
</configuration>

```

## X-Powered-By

在 `web.config` 文件中添加如下代码：

``` XML
<configuration>
    <system.webServer>
        <httpProtocol>
            <customHeaders>
                <remove name="X-Powered-By" />
            </customHeaders>
        </httpProtocol>
    </system.webServer>
</configuration>
```
