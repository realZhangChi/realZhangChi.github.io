---
layout: post
title: "ASP.NET 中的请求验证"
subtitle: 'Request Validation in ASP.NET'
author: "Chi"
date: 2019-07-02 21:13
header-style: text
catalog: true
tags:
  - Design Patterns
---

在进行 Web 开发的时候，我用到了富文本编辑器。我们知道，富文本编辑器是以`html`格式存储内容的。我将富文本的内容作为 Controller 中 Action 的参数（ Model 中的一个字段）来接收的，那么在我运行的时候，通过浏览器的开发人员模式中看到这个请求返回了`error 500`。详细错误信息如下。

``` shell
从客户端(model[description]="<b><u><strike>描述</st...")中检测到有潜在危险的 Request.Form 值。
说明: ASP.NET 在请求中检测到包含潜在危险的数据，因为它可能包括 HTML 标记或脚本。该数据可能表示存在危及应用程序安全的尝试，如跨站点脚本攻击。如果此类型的输入适用于您的应用程序，则可包括明确允许的网页中的代码。有关详细信息，请参阅 http://go.microsoft.com/fwlink/?LinkID=212874。

异常详细信息: System.Web.HttpRequestValidationException: 从客户端(model[description]="<b><u><strike>描述</st...")中检测到有潜在危险的 Request.Form 值。
```

因为我们传送的数据包含`html`标记，所以我们的请求被拒绝了。浏览到网址[Request Validation in ASP.NET](https://docs.microsoft.com/zh-cn/previous-versions/aspnet/hh882339(v=vs.110))找到了解决办法。项目使用的是ASP.NET MVC,所以这里只记录了MVC的解决方案。

## 在 ASP.NET MVC 中禁用请求验证

要在ASP.NET MVC应用程序中禁用请求验证，必须在请求系列发生之前就配置禁用了请求验证。可以在`Web.config`中进行如下配置：

``` xml
<system.web>
  <httpRuntime requestValidationMode="2.0" />
</system.web>
```

在ASP.NET MVC中，我们可以针对 Action 方法、属性或请求中的字段（ input 元素）来禁用请求验证。如果禁用了 Action 方法的请求验证，那么对于任何调用该方法的请求都不进行验证，也就是说，允许所有的输入。因此，针对 Action 方法禁用请求验证是最不安全的。

如果针对属性请求验证，则允许用户针对该属性输入任何值。如果针对特定字段禁用请求验证，则可以控制哪个请求元素（字段）允许任意用户输入。

### 禁用 Action 方法的请求验证

要禁用 Action 方法的请求验证，请使用特性 `ValidateInput(false)`标记该方法，如以下示例所示：

``` C#
[HttpPost]
[ValidateInput(false)]
public ActionResult Edit(string comment)
{
    if (ModelState.IsValid)
    {
        //  Etc.
    }
    return View(comment);
}
```

### 禁用特定属性的请求验证

要禁用特定属性的请求验证，使用`AllowHtml`特性标记属性定义：

``` C#
[AllowHtml]
public string Prop1 { get;  set; }
```

### 禁用特定字段的请求验证

要禁用请求中特定字段的请求验证（例如，对于输入元素或查询字符串值），请在获取项目时调用Request.Unvalidated方法，如以下示例所示：

``` C#
var rawComment = Request.Unvalidated().Form["comment"];
```

## 参考资料

[Request Validation in ASP.NET](https://docs.microsoft.com/en-us/previous-versions/aspnet/hh882339(v=vs.110))
