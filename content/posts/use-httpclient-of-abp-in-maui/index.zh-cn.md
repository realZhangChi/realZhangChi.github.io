+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = ["Abp", "maui"]
date = 2021-10-29T00:15:57Z
description = "在Maui应用程序中使用Abp的Http客户端代理，让网络请求像调用接口一样简单！"
draft = true
tags = ["Abp", "maui"]
title = "如何在Maui中使用Abp的HttpClient"

+++
在移动应用开发中，一般都会涉及到与后端接口进行通讯。如果后端项目采用Abp进行开发，那么可以在Maui中使用Abp的HttpClient来进行接口调用，使Http请求像调用接口一样简单。

## 前提条件

* 本文将讲解在Maui中对于Abp的HttpClient使用方法，有关Abp相关知识请参见Abp官方文档。
* 在Maui中使用Abp的HttpClient依赖于依赖注入系统，在Maui中使用依赖注入请参见“[如何在Maui中使用依赖注入](posts/dependency-injection-in-maui/)”。