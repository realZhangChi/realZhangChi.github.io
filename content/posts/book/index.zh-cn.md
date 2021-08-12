---
title: "Book目录"
date: 2021-07-29T12:01:49+08:00
draft: true
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: ""
tags: ["Abp", "Book"]
categories: ["Book"]
---

## 第一部分 开始

## 第一章 快速入门

1. Abp简介
   简介、基础设施、架构、模块化、cli、帮助方法
2. 模块化
   简介、如何定义模块、配置依赖注入
3. 创建Acme.BookStore
   1. 通过dotnet cli 创建 .net core WebApi应用
   2. 引入nuget依赖，定义模块
   3. 配置Autofac
   4. 运行
4. 总结

### 第六章 应用程序及模块

1. 应用程序
   1. 概念及定义
   2. 功能
   3. 创建
2. 模块
   1. 概念及定义
   2. 生命周期及ModuleLifecycleContributor
   3. Manager

### 第七章 依赖注入与替换

1. **Ioc Principal**简介
2. 服务注册
   1. 约定大于配置
   2. 接口
   3. 特性
3. **DI**
   1. 构造函数
   2. 属性
   3. 方法注入？列权限？
4. 替换默认实现

## 第二部分 架构

目录需调整，按照Abp分层架构来讲，需DDD知识储备，整章围绕整洁架构与依赖倒置开展。
展示层、基础设施层与Interface Adapters的关系

### 第二章 领域层

1. 领域层概念简介
   1. 领域层的关注点
2. 创建Domain.Shard
   1. 项目概述及项目意义
   2. 创建`BookType`枚举
3. 创建Domain
   1. 项目概述及项目意义（通俗介绍or架构理论介绍？）
   2. 创建`Book`聚合
      1. 实体及聚合职责
      2. Abp中的聚合及实体基类
   3. 创建`IBookRepository`
      1. 仓储职责
   4. 创建`DomainService`
      1. 领域服务职责
   5. 创建规约
      1. 意义职责
   6. 创建领域事件
      1. 意义职责
      2. `Book`价格变化
4. 领域层
   1. 作用及意义
   2. 最佳实践
      1. 聚合、实体
      2. 仓储
      3. 领域服务

### 第三章 应用层

1. 应用层概念简介
   1. 应用服务简介
      1. 关注点
2. 创建Application.Contracts
   1. 项目概述及项目意义
   2. 数据传输对象的意义、创建`Dto`
   3. 创建`IBookAppService`
3. 创建Application
   1. 项目概述及项目意义
   2. 创建`BookAppService`
4. 应用层
   1. 定义及意义
   2. 最佳实践

### 第四章 展示层

1. 展示层简介
2. HttpApi和自动控制器
   1. 整洁架构中Adapter
3. HttpApi.Client
   1. 动态代理
   2. AOP
4. 使用`AbpCrudPageBase`创建`Book`页面
   1. 查询
   2. 新增和更新
   3. 删除

### 第五章 基础设施

1. 基础设施层
   1. 职责及关注点
2. EF ORM
   1. 整洁架构中Adapter
      1. 数据库是实现细节
   2. 配置数据迁移及种子数据
   3. 实现仓储
3. DbMigrator项目

### 第六章 组合起来

1. 运行项目
   1. 迁移、Host、Blazor
2. 分层
   1. 为何分层-关注点分离
   2. 关注点分离带来的好处
   3. 单一职责？
   4. 分层依赖
      1. 依赖倒置
3. 整洁架构
   1. 层次结构

## 第三部分 框架

### 第七章 二进制大文件存储

1. Blob简介
   1. BlobProvider
   2. Provider Pattern
      1. Blob模块UML及设计模式分析
2. 为`Book`添加图片
   1. 重构
   2. HttpApi接收图片（Adapter）
      1. 接收文件的最佳实践
   3. 通过`IBlobContainer`保存图片(通过Api还是AppService?)

### 第八章 工作单元及事务

1. 抛出异常
   1. 处理图片时抛出异常，事务回滚
2. 事务控制
   1. 开始事务
      1. 如何开启事务
   2. 自动提交事务
      1. **AOP**
      2. **依赖倒置**
   3. 事务回滚
   4. 多DbContext
      1. Manager
      2. DatabaseApi和TransactionApi
3. 工作单元事件
   1. 保存异常删除图片

### 第九章 验证与授权

1. 使用OAuth2.0
2. 权限系统
   1. 权限定义
      1. Contributor
   2. 角色、用户
      1. 创建`BookStore`员工
3. Identity模块
   1. 角色用户管理
   2. 为角色用户分配权限
      1. 控制`BookStore`员工权限
4. 控制接口访问权限
   1. 基于策略的授权
   2. 基于资源的授权?

### 第十章 多租户

1. 开启多租户
   1. 创建另一个`BookStore`
2. 多租户的数据库
   1. 数据迁移
3. DataFilter
   1. 实现数据权限DataFilter

### 第十一章 本地化功能

1. 国际化的`BookStore`
2. 定义多语言资源
   1. 嵌入式资源
3. 如何使用
   1. 接口
   2. Blazor
4. 跨时区
   1. 时间保存与使用
      1. 最佳实践

### 第十一章 设置

### 第十二章 事件总线

### 第十三章 AOP

## 模块

### Abp的应用模块


### 第八章 AOP

1. **AOP**概念简介
2. 动态代理，Interface介绍及框架中的AOP，**适配器模式**
3. 实现AOP
4. 自定义返回值（Filter？）

### 第九章 多租户

### AspNetCore、AspNetCoreMvc

XSRF/CSRF/Anti-Forgery、CorrelationId、UOW

### 第三章 身份验证

将在Abp 5.0发布后开始

### 第四章 授权与权限管理

### 第五章 多租户系统

1. 配置与使用多租户
2. 多租户数据迁移

### 第六章 动态代理

## 第三部分 模块化

框架模块和应用模块

设置模块，

讲store，讲模块化生命周期，讲Options模式，讲ValueProvider模式，讲空对象模式，
Setting模块，Provider Pattern，Bridge Pattern，factory and abstract pattern, 策略模式

讲对象扩展

讲数据provider，功能Contributor，数据Accessor

讲更新时并发时间戳问题

实战，利用DataFilter实现数据权限

讲乐观并发？

TODO： 依赖注入尽早讲
TODO： 依赖倒置尽早讲