+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = ["ASP.NET Core"]
date = 2021-12-22T04:33:11Z
description = "配置在开发环境中使用HTTP而不是HTTPS，配置不同的端口号。"
tags = ["ASP.NET Core"]
title = "禁用ASP.NET Core开发环境的HTTPS"

+++
在开发ASP.NET Core的项目时，默认地是使用HTTPS安全协议的。有时候可能不希望在本地的开发环境中使用HTTPS，更改这一默认行为非常简单。

1. 移除`UseHttpsRedirection`中间件

   `app.UseHttpsRedirection`中间件会将所有HTTP请求重定向到HTTPS，因此首先我们需要将其删除。中间件配置一般在Program.cs或Startup.cs中。
2. 配置launchSettings.json

   launchSettings.json在项目的Properties目录下，它只对本地的**开发环境**生效，部署时会被忽略。通过`dotnet new`或者Visual Studio生成的ASP.NET Core项目会创建launchSettings.json文件。

       {
         "iisSettings": {
           "windowsAuthentication": false,
           "anonymousAuthentication": true,
           "iisExpress": {
             "applicationUrl": "http://localhost:16717",
             "sslPort": 44324
           }
         },
         "profiles": {
           "WebApplication1": {
             "commandName": "Project",
             "dotnetRunMessages": true,
             "launchBrowser": true,
             "applicationUrl": "https://localhost:7072;http://localhost:5072",
             "environmentVariables": {
               "ASPNETCORE_ENVIRONMENT": "Development"
             }
           },
           "IIS Express": {
             "commandName": "IISExpress",
             "launchBrowser": true,
             "environmentVariables": {
               "ASPNETCORE_ENVIRONMENT": "Development"
             }
           }
         }
       }

   将`applicationUrl`从https更改为http即可更改默认的应用启动Url，若使用IIS启动，还需将`iisSettings`中的`sslPort`设为0。

       {
         "iisSettings": {
           "windowsAuthentication": false,
           "anonymousAuthentication": true,
           "iisExpress": {
             "applicationUrl": "http://localhost:16717",
             "sslPort": 0
           }
         },
         "profiles": {
           "WebApplication1": {
             "commandName": "Project",
             "dotnetRunMessages": true,
             "launchBrowser": true,
             "applicationUrl": "http://localhost:5072",
             "environmentVariables": {
               "ASPNETCORE_ENVIRONMENT": "Development"
             }
           },
           "IIS Express": {
             "commandName": "IISExpress",
             "launchBrowser": true,
             "environmentVariables": {
               "ASPNETCORE_ENVIRONMENT": "Development"
             }
           }
         }
       }

此外，在launchSettings.json中，也可以通过`applicationUrl`更改应用启动的端口号。