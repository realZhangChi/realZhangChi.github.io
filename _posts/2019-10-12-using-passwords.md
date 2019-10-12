---
layout: post
title: "Identity Server 4 教程 Part 2: 使用密码保护API"
subtitle: 'Identity Server 4 Quickstarts Part 2: Protecting an API using Passwords'
author: "Chi"
date: 2019-10-12 10:46
header-style: text
catalog: true
tags:
  - Identity Server 4
  - Quickstarts
---

## 前言

这篇文章是紧接着 [Identity Server 4 教程 Part 1: 使用客户端凭据保护API](https://blog.zhangchi.fun/2019/09/27/using-client-credentials/) 的Identity Server 4系列文章的第二部分。这里涉及到的一些代码，是在上一篇文章的基础之上进行了修改。

我们可以通过[IdentityServer4 repository]找到官方示例源码，也可以访问[我的 github repository](https://github.com/realZhangChi/IdentityServer4)来获取我在学习过程中的实践代码。

OAuth 2.0 resource owner password grant允许客户端将用户名和密码发送到令牌服务（token service），并获得代表该用户的访问令牌（access token）。

## 添加用户

就像在内存中存储资源（即scopes）和客户端（client）一样，我们也可以来针对用户来进行设置。`TestUser`类代表测试用户及其声明（claims）。我们通过将以下代码添加到配置类（`Config.cs`）中来创建几个用户：

``` C#
using IdentityServer4.Test;

public static List<TestUser> GetUsers()
{
    return new List<TestUser>
    {
        new TestUser
        {
            SubjectId = "1",
            Username = "alice",
            Password = "password"
        },
        new TestUser
        {
            SubjectId = "2",
            Username = "bob",
            Password = "password"
        }
    };
}
```

然后将这些用户注册到IdentityServer中：

``` C#
public void ConfigureServices(IServiceCollection services)
{
    // configure identity server with in-memory stores, keys, clients and scopes
    services.AddIdentityServer()
        .AddInMemoryApiResources(Config.GetApiResources())
        .AddInMemoryClients(Config.GetClients())
        .AddTestUsers(Config.GetUsers());
}
```

`AddTestUsers`扩展方法，做了以下工作：

- 为 resource owner password grant 添加支持
- 为用户相关服务（用户登录界面会使用，在下一节教程中会介绍）提供支持
- 根据测试用户添加个人资料服务（profile service）的支持（在下一节教程中会介绍）

## 为resource owner password grant添加客户端

我们将为资源所有者创建一个单独的客户端：

``` C#
public static IEnumerable<Client> GetClients()
{
    return new List<Client>
    {
        // other clients omitted...

        // resource owner password grant client
        new Client
        {
            ClientId = "ro.client",
            AllowedGrantTypes = GrantTypes.ResourceOwnerPassword,

            ClientSecrets =
            {
                new Secret("secret".Sha256())
            },
            AllowedScopes = { "api1" }
        }
    };
}
```

## 使用密码来请求访问令牌

更改上一节中我们创建的`Client`项目，将通过客户端凭据来请求令牌的代码，替换成通过密码来请求令牌：

``` C#
// request token using Passwords
var tokenResponse = await client.RequestPasswordTokenAsync(new PasswordTokenRequest
{
    Address = disco.TokenEndpoint,
    ClientId = "ro.client",
    ClientSecret = "secret",

    UserName = "alice",
    Password = "password",
    Scope = "api1"
});
```

与通过客户端凭据访问API不同的是，当我们携带通过密码获取的访问令牌（access token）来访问`identity`API时，会发现我们的访问令牌（access token）现在包含了`sub`声明，这个`sub`声明唯一区分了不同的用户。可以通过在调用API之后检查内容变量来查看此`sub`声明，并且控制台应用程序还将其显示在屏幕上。

`sub`声明的是否存在，使API可以区分出来代表客户端的调用和代表用户的调用。

## 参考

> [Protecting an API using Passwords](http://docs.identityserver.io/en/latest/quickstarts/2_resource_owner_passwords.html)
