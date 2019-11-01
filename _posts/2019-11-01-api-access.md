---
layout: post
title: "Identity Server 4 教程: Part 4 用户验证和API访问"
subtitle: 'Identity Server 4 Quickstarts Part 4: ASP.NET Core and API access'
author: "Chi"
date: 2019-11-01 14:00
header-style: text
catalog: true
tags:
  - Identity Server 4
  - Quickstarts
---

## 前言

这篇文章是紧接着 [Identity Server 4 教程: Part 3 添加ASP.NET Core交互应用程序](https://blog.zhangchi.fun/2019/10/31/user-authentication/) 的Identity Server 4系列文章的第四部分。这里涉及到的一些代码，是在前几篇文章的基础之上进行了修改。

我们可以通过[IdentityServer4 repository]("https://github.com/IdentityServer/IdentityServer4/blob/master/samples")找到官方示例源码，也可以访问[我的 github repository](https://github.com/realZhangChi/IdentityServer4)来获取我在学习过程中的实践代码。

在前面的文章中，我们探索了用户身份验证和API访问控制。现在我们将把这两部分结合起来。

组合使用OpenID Connect和OAuth 2.0的好处在于，我们可以只使用一种协议并且只访问一次令牌（token）服务器，来同时实现用户身份验证和API访问控制。

到目前为止，我们仅在令牌请求期间要求提供身份资源，当我们还要求提供API资源的时候，IdentityServer将返回两个令牌：包含有关身份验证和会话信息的身份令牌，以及代表登录用户访问API的访问令牌。

## 修改客户端配置

在IdentityServer中更新客户端配置非常简单，我们只需要将`api1`资源添加到允许的作用域列表中即可。此外，我们还通过`AllowOfflineAccess`属性启用对刷新令牌的支持：

``` C#
new Client
{
    ClientId = "mvc",
    ClientSecrets = { new Secret("secret".Sha256()) },

    AllowedGrantTypes = GrantTypes.Code,
    RequireConsent = false,
    RequirePkce = true,

    // where to redirect to after login
    RedirectUris = { "http://localhost:5002/signin-oidc" },

    // where to redirect to after logout
    PostLogoutRedirectUris = { "http://localhost:5002/signout-callback-oidc" },

    AllowedScopes = new List<string>
    {
        IdentityServerConstants.StandardScopes.OpenId,
        IdentityServerConstants.StandardScopes.Profile,
        "api1"
    },

    AllowOfflineAccess = true
}
```

## 修改MVC客户端

MVC客户端现在要做的就是通过scope参数请求其他资源。我们在`Startup.cs`的OpenID Connect处理程序配置中做一些更改：

``` C#
services.AddAuthentication(options =>
{
    options.DefaultScheme = "Cookies";
    options.DefaultChallengeScheme = "oidc";
})
    .AddCookie("Cookies")
    .AddOpenIdConnect("oidc", options =>
    {
        options.Authority = "http://localhost:5000";
        options.RequireHttpsMetadata = false;

        options.ClientId = "mvc";
        options.ClientSecret = "secret";
        options.ResponseType = "code";

        options.SaveTokens = true;

        options.Scope.Add("api1");
        options.Scope.Add("offline_access");
    });
```

由于启用了`SaveTokens`，因此ASP.NET Core将在身份验证会话（session ）中自动存储访问结果并刷新令牌。我们可以在页面上看到这些数据。

## 使用访问令牌（access token）

我们可以使用可以在Microsoft.AspNetCore.Authentication命名空间中的扩展方法来获取会话中的令牌：

``` C#
var accessToken = await HttpContext.GetTokenAsync(“access_token”) v
```

要通过访问令牌（access token）来访问API，我们需要做的就是获取令牌，并将其设置在HttpClient上：

``` C#
public async Task<IActionResult> CallApi()
{
    var accessToken = await HttpContext.GetTokenAsync("access_token");

    var client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    var content = await client.GetStringAsync("http://localhost:5001/identity");

    ViewBag.Json = JArray.Parse(content).ToString();
    return View("json");
}
```

创建一个名为json.cshtml的视图，该视图将输出我们获取到的json数据：

``` cshtml
<pre>@ViewBag.Json</pre>
```

运行IdentityServer和Api项目，然后运行MvcClient项目，并在登录之后访问`/home/CallApi`。

## 相关章节

[Identity Server 4 教程: Part 3 添加ASP.NET Core交互应用程序](https://blog.zhangchi.fun/2019/10/31/user-authentication/)

## 参考

> [ASP.NET Core and API access](http://docs.identityserver.io/en/latest/quickstarts/3_aspnetcore_and_apis.html)
