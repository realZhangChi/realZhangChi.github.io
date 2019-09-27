---
layout: post
title: "Identity Server 4: 术语"
subtitle: 'Identity Server 4: Terminology'
author: "Chi"
date: 2019-09-25
header-style: text
catalog: false
tags:
  - Identity Server 4
---

以前一直对身份验证过程中的一些术语的含义及其作用不明白，在阅读Identityt Server 4文档的时候，发现对一些术语含义的介绍很简洁明了，在此翻译记录下来。

## IdentityServer

![terminology]("/img/in-post/2019-09-27-identity-server-4-terminology/terminology.png")

IdentityServer 是 OpenID Connect 提供程序，它实现了OpenID Connect 和 OAuth 2.0 协议。

不同的文献对同一角色使用不同的术语，在其他地方我们可能会见到security token service, identity provider, authorization server, IP-STS等。

简而言之，它们都是一样的：一种向客户端颁发安全令牌的软件。

IdentityServer具有许多功能：

- 保护系统资源
- 使用本地帐户数据或通过外部身份提供商(如Google, Facebook)对用户进行身份验证
- 提供会话管理和单点登录
- 向客户端发布身份验证和访问令牌
- 验证令牌

## User

用户是使用在IdentityServer中注册的客户端来访问资源的人。

## Client

客户端是软件中从IdentityServer请求令牌的部分，用于认证用户（请求身份令牌）或用于访问资源（请求访问令牌）。客户端必须先向IdentityServer注册，然后才能请求令牌。

客户端的示例包括Web应用程序，本地手机APP或桌面应用程序，SPA，服务器进程等。

## Resources

资源是要使用IdentityServer保护的东西，比如用户的身份数据(Identity data)或API。

每个资源都有一个唯一的名称-客户端使用此名称来指定他们要访问的资源。

**身份数据(Identity data)**:有关用户的身份信息（也称为声明claims），例如名称或电子邮件地址。
**APIs**: API资源代表客户端想要调用的功能,通常实现为Web API。

## Identity Token

身份令牌(Identity Token)表示身份验证过程的结果,它至少包含用户的标识符(又称sub, subject claim)，以及有关用户如何以及何时进行认证的信息。它可以包含其他身份数据。

## Access Token

访问令牌(Access Token)允许客户端访问API资源。客户端请求访问令牌并携带访问令牌来访问API。访问令牌包含有关客户端和用户（如果有）的信息。API使用该信息来授权对其数据的访问。
