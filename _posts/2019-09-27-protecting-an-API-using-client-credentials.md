---
layout: post
title: "Identity Server 4 Quickstart: Part 1 使用客户端凭据保护API"
subtitle: 'Identity Server 4 Quickstart: Part 1  Protecting an API using Client Credentials'
author: "Chi"
date: 2019-09-27 14:40
header-style: text
catalog: false
tags:
  - Identity Server 4
  - Quickstart
---

## 源代码

与所有这些快速入门一样，可以在[IdentityServer4 repository]("https://github.com/IdentityServer/IdentityServer4/blob/master/samples")中找到它的源代码。
此快速入门的项目是[Quickstart #1: Securing an API using Client Credentials]("https://github.com/IdentityServer/IdentityServer4/tree/master/samples/Quickstarts/1_ClientCredentials")

## 准备

要安装自定义模板，请打开控制台窗口，然后输入以下命令：

``` Powershell
dotnet new -i IdentityServer4.Templates
```

## 初始化ASP.NET Core 应用

首先为应用程序创建一个目录，然后使用我们的模板创建一个包含基本IdentityServer安装程序的ASP.NET Core应用程序，例如：

``` Powershell
md quickstart
cd quickstart

md src
cd src

dotnet new is4empty -n IdentityServer
```

这将创建以下文件:

![file](/img/2019-09-27-protecting-an-API-using-client-credentials/file.png)

- `IdentityServer.csproj`-项目文件和`Properties\launchSettings.json`文件
- `Program.cs`和`Startup.cs`-主应用程序入口点
- `Config.cs`-IdentityServer资源和客户端配置文件

如果要获得Visual Studio支持，可以添加如下解决方案文件：

``` Powershell
cd ..
dotnet new sln -n Quickstart
```

并添加您的IdentityServer项目（请牢记此命令，因为我们将在下面创建其他项目）:

``` Powershell
dotnet sln add .\src\IdentityServer\IdentityServer.csproj
```

## 定义API资源

API是系统中要保护的资源。资源定义可以通过多种方式加载，上面用于创建项目的模板显示了如何使用“代码作为配置”方法。

找到Config.cs文件，您可以找到一个名为GetApis的方法，定义API如下：

``` C#
public static IEnumerable<ApiResource> GetApis()
{
    return new List<ApiResource>
    {
        new ApiResource("api1", "My API")
    };
}
```

> 如果在生产环境中使用它，那么给API取一个逻辑名称就很重要。开发人员将使用它通过身份服务器连接到API。它应该用简单的术语向开发人员和用户描述API。

一个很好的例子是：

``` C#
new ApiResource("afcpayroll", "Acme Fireworks Co. payroll")
```

不建议的用法：

``` C#
new ApiResource("testapi", "My first api")
```

## 定义客户端

下一步是定义将用于访问新API的客户端应用程序。

在这里客户端将没有交互的用户，并且将通过IdentityServer使用客户端密钥进行身份验证。

再次打开Config.cs文件，并向其中添加以下代码：

``` C#
public static IEnumerable<Client> GetClients()
{
    return new List<Client>
    {
        new Client
        {
            ClientId = "client",

            // no interactive user, use the clientid/secret for authentication
            AllowedGrantTypes = GrantTypes.ClientCredentials,

            // secret for authentication
            ClientSecrets =
            {
                new Secret("secret".Sha256())
            },

            // scopes that client has access to
            AllowedScopes = { "api1" }
        }
    };
}
```

可以将`ClientId`和`ClientSecret`视为应用程序本身的登录名和密码。它将您的应用程序标识到身份服务器，以便它知道哪个应用程序正在尝试与其连接。

## 配置IdentityServer

资源和客户端定义是在Startup.cs中加载的-模板已经完成了此操作：

``` C#
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddIdentityServer()
        .AddInMemoryIdentityResources(Config.GetIdentityResources())
        .AddInMemoryApiResources(Config.GetApis())
        .AddInMemoryClients(Config.GetClients());

    // omitted for brevity
}
```

IdentityServer就是按照上述代码所示进行配置的。如果运行服务器并在浏览器中导航到<http://localhost:5000/.well-known/openid-configuration>，则应该看到IdentityServer文档。

![discovery](/img/2019-09-27-protecting-an-API-using-client-credentials/1_discovery.png)

首次启动时，IdentityServer将创建一个开发人员签名密钥，该文件名为tempkey.rsa。

![tempkey](/img/2019-09-27-protecting-an-API-using-client-credentials/tempkey.png)

无需将该文件签入源代码管理中，如果不存在该文件将被重新创建。

## 添加API

接下来在解决方案中添加一个API项目。

在`src`目录下执行命令：

``` Powershell
dotnet new web -n Api
```

然后通过运行以下命令将其添加到解决方案中：

``` Powershell
cd ..
dotnet sln add .\src\Api\Api.csproj
```

将API应用程序配置为仅在<http://localhost:5001>上运行。可以通过编辑`Properties`文件夹中的`launchSettings.json`文件来实现。将应用程序URL设置更改为：

``` json
"applicationUrl": "http://localhost:5001"
```

### 控制器

添加文件夹`Controllers`和控制器`IdentityController`（如果使用的是Visual Studio，请在API项目中选择`API controller empty`）：

``` C#
[Route("identity")]
[Authorize]
public class IdentityController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return new JsonResult(from c in User.Claims select new { c.Type, c.Value });
    }
}
```

稍后将使用此控制器来测试授权要求，并通过API来将身份声明(claims identity)可视化。

### 配置

最后一步是将身份验证服务添加到DI（依赖注入），并将身份验证中间件添加到管道。这将：

- 验证传入令牌以确保它来自受信任的发行者
- 验证令牌是否可以与此API一起使用（也称audience）

更改 *Startup* 文件:

``` C#
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvcCore()
            .AddAuthorization()
            .AddJsonFormatters();

        // If ASP.NET Core 3.0
        // services.AddMvcCore(option =>
        //     {
        //         option.EnableEndpointRouting = false;
        //     })
        //         .AddAuthorization();

        services.AddAuthentication("Bearer")
            .AddJwtBearer("Bearer", options =>
            {
                options.Authority = "http://localhost:5000";
                options.RequireHttpsMetadata = false;

                options.Audience = "api1";
            });
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseAuthentication();

        app.UseMvc();
    }
}
```

`AddAuthentication`将身份验证服务添加到DI并将`“Bearer”`配置为默认方案。`UseAuthentication`将身份验证中间件添加到管道中，因此将在每次对主机的调用中自动执行身份验证。

在浏览器上导航到控制器`http://localhost:5001/identity` 应该返回401状态代码。这意味着访问此API需要凭据，并且现在受IdentityServer保护。

## 添加客户端
