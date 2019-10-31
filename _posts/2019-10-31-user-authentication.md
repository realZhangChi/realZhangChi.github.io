---
layout: post
title: "Identity Server 4 教程: Part 3 添加ASP.NET Core交互应用程序"
subtitle: 'Identity Server 4 Quickstarts Part 3: Interactive Applications with ASP.NET Core'
author: "Chi"
date: 2019-10-31 14:22
header-style: text
catalog: true
tags:
  - Identity Server 4
  - Quickstarts
---

## 前言

这篇文章是紧接着 [Identity Server 4 教程 Part 2: 使用密码保护API](https://blog.zhangchi.fun/2019/10/12/using-passwords/) 的Identity Server 4系列文章的第三部分。这里涉及到的一些代码，是在上一篇文章的基础之上进行了修改。

我们可以通过[IdentityServer4 repository]("https://github.com/IdentityServer/IdentityServer4/blob/master/samples")找到官方示例源码，也可以访问[我的 github repository](https://github.com/realZhangChi/IdentityServer4)来获取我在学习过程中的实践代码。

在本快速入门中，我们将通过OpenID Connect协议添加交互式用户身份验证的界面支持到上一章中构建的IdentityServer中。我们将创建一个将使用IdentityServer进行身份验证的MVC应用程序。

## 添加UI

Identity Server已内置OpenID Connect所需的所有协议支持。我们需要为其提供登录，注销等所需的UI部分。

我们可以使用.NET CLI添加界面支持（从src / IdentityServer文件夹中运行）：

``` Powershell
dotnet new is4ui
```

添加MVC UI后，还需要在DI系统和管道中启用MVC。我们查看Startup.cs时，会在ConfigureServices和Configure方法中找到如何启用MVC的注释。

> 需要注意的是，如果我们基于ASP.NET Core 3.0来进行开发，那么需要在`ConfigureServices`中使用`services.AddControllersWithViews();`，同时在`Configure`方法中也需进行更改。详细变更可以参见我的[代码仓库](https://github.com/realZhangChi/IdentityServer4)。

运行`IdentityServer`项目，我们将看到新添加的界面主页。

## 创建MVC客户端

接下来需要创建一个MVC客户端项目，并将其配置在5002端口上运行。此外，需要将包含OpenID Connect处理程序的nuget包添加到项目中，以支持使用OpenID Connect身份验证：

``` Powershell
dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect
```

在`Startup`的`ConfigureServices`方法中，注册身份验证支持:

``` C#
JwtSecurityTokenHandler.DefaultMapInboundClaims = false;

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
    });
```

1. `AddAuthentication`将身份验证服务添加到了DI。

2. 我们使用Cookie来本地登录用户（将`“Cookies”`作为`DefaultScheme`），并且将`DefaultChallengeScheme`设置为`oidc`，因为当处理用户登录时，我们使用的是OpenID Connect协议。

3. 然后，我们使用`AddCookie`添加可以处理cookies的处理程序。

4. 最后，`AddOpenIdConnect`用于配置执行OpenID Connect协议的处理程序。`Authority`指受信任令牌服务的位置。然后，我们通过`ClientId`和`ClientSecret`来标明此客户端。`SaveTokens`用于将IdentityServer中的令牌保留在cookie中（稍后将会使用）。

要确保身份验证服务对每个请求都会执行，在`Startup`中的`Configure`添加`UseAuthentication`：

``` C#
app.UseStaticFiles();

app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.UseEndpoints(endpoints =>
{
    endpoints.MapDefaultControllerRoute()
        .RequireAuthorization();
});
```

> `RequireAuthorization`方法禁用整个应用程序的匿名访问。如果要基于每个控制器或操作方法进行指定，也可以使用`[Authorize]`特性。

修改主视图以显示用户的声明以及cookie属性：

``` cshtml
@using Microsoft.AspNetCore.Authentication

<h2>Claims</h2>

<dl>
    @foreach (var claim in User.Claims)
    {
        <dt>@claim.Type</dt>
        <dd>@claim.Value</dd>
    }
</dl>

<h2>Properties</h2>

<dl>
    @foreach (var prop in (await Context.AuthenticateAsync()).Properties.Items)
    {
        <dt>@prop.Key</dt>
        <dd>@prop.Value</dd>
    }
</dl>
```

如果现在使用浏览器打开这个项目的页面，将尝试重定向到IdentityServer-这将导致错误，因为尚未注册MVC客户端。

## 添加对OpenID Connect Identity Scope的支持

与OAuth 2.0相似，OpenID Connect也使用了“scopes”这个概念。类似的，“scopes”代表我们要保护的内容也就是客户端要访问的内容。与OAuth不同的是，OIDC中的“scopes”不代表API，而是身份数据，例如用户ID，名称或电子邮件地址。

通过修改`Config.cs`中的`Ids`属性，添加对标准`openid`（subject id）和`profile`（名字，姓氏等）“scopes”的支持：

``` C#
public static IEnumerable<IdentityResource> Ids =>
    new List<IdentityResource>
    {
        new IdentityResources.OpenId(),
        new IdentityResources.Profile(),
    };
```

在`Startup.cs`中向IdentityServer注册身份资源：

``` C#
var builder = services.AddIdentityServer()
    .AddInMemoryIdentityResources(Config.Ids)
    .AddInMemoryApiResources(Config.Apis)
    .AddInMemoryClients(Config.Clients);
```

## 添加测试用户

示例UI还带有一个内存“用户数据库”。可以通过添加`AddTestUsers`扩展方法在IdentityServer中启用它：

``` C#
var builder = services.AddIdentityServer()
    .AddInMemoryIdentityResources(Config.Ids)
    .AddInMemoryApiResources(Config.Apis)
    .AddInMemoryClients(Config.Clients)
    .AddTestUsers(TestUsers.Users);
```

导航到TestUsers类时，您会看到已定义了两个名为alice和bob的用户以及一些身份声明。我们可以使用这些用户登录。

## 将MVC客户端添加到IdentityServer配置

最后一步是将MVC客户端的新配置项添加到IdentityServer。

基于OpenID Connect的客户端与我们添加的OAuth 2.0客户端非常相似。但是，由于OIDC中的数据传输流始终是交互式的，因此我们需要在配置中添加一些重定向URL。

客户端列表应如下所示：

``` C#
public static IEnumerable<Client> Clients =>
    new List<Client>
    {
        // machine to machine client (from quickstart 1)
        new Client
        {
            ClientId = "client",
            ClientSecrets = { new Secret("secret".Sha256()) },

            AllowedGrantTypes = GrantTypes.ClientCredentials,
            // scopes that client has access to
            AllowedScopes = { "api1" }
        },
        // interactive ASP.NET Core MVC client
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
                IdentityServerConstants.StandardScopes.Profile
            }
        }
    };
```

## 测试客户端

导航到受保护的页面将触发身份认证，这时网页应当重定向到IdentityServer上的登录页面。

![login](/img/in-post/2019-10-31-user-authentication/login.png)

登录之后，IdentityServer将重定向回MVC客户端，在该客户端上，OpenID Connect身份验证处理程序将处理响应，并通过设置cookie在本地保存登录用户的信息。

最后，MVC视图将显示cookie的内容。

coming soon...
