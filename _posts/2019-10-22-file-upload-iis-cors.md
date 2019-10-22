---
layout: post
title: "记一次上传文件到IIS服务器，提示跨域错误的经历"
author: "Chi"
date: 2019-10-22 15:19
header-style: text
catalog: false
---

在一个前后分离的项目中，前端Angular使用 `ng2-file-upload` 调用后端 ASP.NET Core 2.2 Webapi 项目上传文件到 `wwwroot` 文件夹下。

后端项目跨域配置确定正确，其他接口前端项目访问是没有问题的，但是在上传文件的时候，浏览器 `console` 中输出了跨域的错误信息。查看 `Network` 记录，调用接口的请求返回错误代码 `500`。

根据跨域查找了大概一天的解决方案，调整了多次跨域配置，后来发现，项目发布在本地IIS的时候时没有问题的，在服务端就不行，确定这是服务端IIS配置问题，而不是跨域问题，于是把问题的重点放在服务器返回的错误代码 `500` 上，继续Goole......

最终发现了问题所在竟是文件夹的读写权限问题。右键 `wwwroot` 目录，在属性的安全选项卡中，编辑用户权限，将 `IUSER` 和 `USER` 的权限改为完全控制，同时将 `wwwroot` 中上传文件的子目录也做同样修改，解决了问题。

权限不足却返回跨域问题的原因在于，服务器执行代码出了错误（错误代码 `500`），没有将跨域所需的响应头 `Access-Control-Allow-Origin`、`Access-Control-Allow-Methods`、`Access-Control-Allow-Headers` 返回给浏览器，从而导致浏览器判断跨域请求失败。
