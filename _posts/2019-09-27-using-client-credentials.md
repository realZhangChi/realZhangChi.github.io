---
layout: post
title: "Identity Server 4 教程 Part 1: 使用客户端凭据保护API"
subtitle: 'Identity Server 4 Quick Start Part 1: Protecting an API using Client Credentials'
author: "Chi"
date: 2019-09-27 14:40
header-style: text
catalog: true
tags:
  - Identity Server 4
  - Quick Start
---

## 源代码

可以在[IdentityServer4 repository]("https://github.com/IdentityServer/IdentityServer4/blob/master/samples")中找到它的源代码。
此快速入门的项目是[Quickstart #1: Securing an API using Client Credentials]("https://github.com/IdentityServer/IdentityServer4/tree/master/samples/Quick Start/1_ClientCredentials")。

也可以访问[我的 github repository](https://github.com/realZhangChi/IdentityServer4)来获取我在学习过程中的实践代码。

## 准备

要安装自定义模板，请打开控制台窗口，然后输入以下命令：

``` Powershell
dotnet new -i IdentityServer4.Templates
```

## 初始化ASP. NET Core 应用

首先为应用程序创建一个目录，然后使用我们的模板创建一个包含基本IdentityServer安装程序的ASP. NET Core应用程序，例如：

``` Powershell
md quickstart
cd quickstart

md src
cd src

dotnet new is4empty -n IdentityServer
```

这将创建以下文件:

![file](/img/in-post/2019-09-27-protecting-an-api/file.png)

* `IdentityServer.csproj` -项目文件和 `Properties\launchSettings.json` 文件
* `Program.cs` 和 `Startup.cs` -主应用程序入口点
* `Config.cs` -IdentityServer资源和客户端配置文件

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

可以将 `ClientId` 和 `ClientSecret` 视为应用程序本身的登录名和密码。它将您的应用程序标识到身份服务器，以便它知道哪个应用程序正在尝试与其连接。

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

IdentityServer就是按照上述代码所示进行配置的。如果运行服务器并在浏览器中导航到<http://localhost:5000/.well-known/openid-configuration>，则应该看到IdentityServer发现文档(discovery document)。

![discovery](/img/in-post/2019-09-27-protecting-an-api/1_discovery.png)

首次启动时，IdentityServer将创建一个开发人员签名密钥，该文件名为tempkey.rsa。

![tempkey](/img/in-post/2019-09-27-protecting-an-api/tempkey.png)

无需将该文件签入源代码管理中，如果不存在该文件将被重新创建。

## 添加API

接下来在解决方案中添加一个API项目。

在 `src` 目录下执行命令：

``` Powershell
dotnet new web -n Api
```

然后通过运行以下命令将其添加到解决方案中：

``` Powershell
cd ..
dotnet sln add .\src\Api\Api.csproj
```

将API应用程序配置为仅在<http://localhost:5001>上运行。可以通过编辑 `Properties` 文件夹中的 `launchSettings.json` 文件来实现。将应用程序URL设置更改为：

``` json
"applicationUrl": "http://localhost:5001"
```

### 控制器

添加文件夹 `Controllers` 和控制器 `IdentityController` （如果使用的是Visual Studio，请在API项目中选择 `API controller empty` ）：

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

* 验证传入令牌以确保它来自受信任的发行者
* 验证令牌是否可以与此API一起使用（也称audience）

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

`AddAuthentication` 将身份验证服务添加到DI并将 `“Bearer”` 配置为默认方案。 `UseAuthentication` 将身份验证中间件添加到管道中，因此将在每次对主机的调用中自动执行身份验证。

在浏览器上导航到控制器 `http://localhost:5001/identity` 应该返回401状态代码。这意味着访问此API需要凭据，并且现在受IdentityServer保护。

## 添加客户端

最后一步是编写一个请求访问令牌的客户端，然后使用该令牌访问API。在 `src` 目录下执行以下命令来添加一个Client项目：

``` Powershell
dotnet new console -n Client

cd ..
dotnet sln add .\src\Client\Client.csproj
```

### 全部代码

打开 `Program.cs` 文件并进行编辑：

``` C#
using System. Net. Http;
using System;
using System. Threading. Tasks;
using IdentityModel. Client;
using Newtonsoft. Json. Linq;

namespace Client
{

    class Program
    {
        private static async Task Main()
        {
            // discover endpoints from metadata
            var client = new HttpClient();

            var disco = await client.GetDiscoveryDocumentAsync("http://localhost:5000");
            if (disco.IsError)
            {
                Console.WriteLine(disco.Error);
                return;
            }

            // request token
            var tokenResponse = await client.RequestClientCredentialsTokenAsync(new ClientCredentialsTokenRequest
            {
                Address = disco.TokenEndpoint,
                ClientId = "client",
                ClientSecret = "secret",

                Scope = "api1"
            });

            if (tokenResponse.IsError)
            {
                Console.WriteLine(tokenResponse.Error);
                return;
            }

            Console.WriteLine(tokenResponse.Json);
            Console.WriteLine("\n\n");

            // call api
            var apiClient = new HttpClient();
            apiClient.SetBearerToken(tokenResponse.AccessToken);

            var response = await apiClient.GetAsync("http://localhost:5001/identity");
            if (!response.IsSuccessStatusCode)
            {
                Console.WriteLine(response.StatusCode);
            }
            else
            {
                var content = await response.Content.ReadAsStringAsync();
                Console.WriteLine(JArray.Parse(content));
            }
        }
    }

}

```

客户端程序异步调用 `Main` 方法，以运行异步http调用。异步调用 `Main` 方法从 `C# 7.1` 开始支持。将以下代码添加到 `Client.csproj` 的 `PropertyGroup` 节点中以使用最新版本的C#：

``` csproj
<LangVersion>latest</LangVersion>
```

IdentityServer上的令牌端点实现了OAuth 2.0协议，我们可以使用原始HTTP来访问它。但是，通过使用 `IdentityModel` 客户端库，可以更方便的进行协议交互。将 `IdentityModel` Nuget包添加到Client项目中，在Client项目路径下执行以下命令：

``` Powershell
dotnet add package IdentityModel
```

### 访问IdentityServer发现文档(discovery document)

IdnetityModel可以根据基地址从元数据中自动查找IdentityServer发现文档(discovery document)端点地址：

``` C#
// discover endpoints from metadata
var client = new HttpClient();
var disco = await client. GetDiscoveryDocumentAsync("http://localhost:5000");
if (disco. IsError)
{

    Console.WriteLine(disco.Error);
    return;

}

```

### 请求token令牌

接下来，可以使用IdentityServer发现文档(discovery document)中的信息向IdentityServer请求令牌以访问 `api1` ：

``` C#
// request token
var tokenResponse = await client.RequestClientCredentialsTokenAsync(new ClientCredentialsTokenRequest
{
    Address = disco.TokenEndpoint,

    ClientId = "client",
    ClientSecret = "secret",
    Scope = "api1"
});

if (tokenResponse.IsError)
{
    Console.WriteLine(tokenResponse.Error);
    return;
}

Console.WriteLine(tokenResponse.Json);
```

其中 `ClientId` 、 `ClientSecret` 和 `Scope` 已在 `IdentityServer` 项目中的 `Config.cs` 文件中定义。

### 使用token令牌访问Api

要将访问令牌发送到API，通常使用HTTP授权标头。这是使用SetBearerToken扩展方法完成的：

``` C#
var client = new HttpClient();
client. SetBearerToken(tokenResponse. AccessToken);

var response = await client. GetAsync("http://localhost:5001/identity");
if (!response. IsSuccessStatusCode)
{

    Console.WriteLine(response.StatusCode);

}
else
{

    var content = await response.Content.ReadAsStringAsync();
    Console.WriteLine(JArray.Parse(content));

}

```

## 相关章节

[Identity Server 4 教程 Part 2: 使用密码保护API](https://blog.zhangchi.fun/2019/10/12/using-passwords/)

## 参考

> [Protecting an API using Client Credentials](http://docs.identityserver.io/en/latest/Quick Start/1_client_credentials.html#protecting-an-api-using-client-credentials)
