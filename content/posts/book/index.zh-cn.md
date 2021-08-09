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
3. 创建控制台Acme.BookStore
   1. 通过dotnet cli 创建 .net core 控制台应用
   2. 引入nuget依赖，定义模块
   3. 配置依赖注入
   4. 运行
4. 总结

## 第二部分 架构

目录需调整，按照Abp分层架构来讲，需DDD知识储备，整章围绕整洁架构与依赖倒置开展。
展示层、基础设施层与Interface Adapters的关系

### 第二章 整洁架构

1. 关注点分离、项目依赖原则
2. 四层简介
3. 控制流程与依赖倒置

### 第三章 DDD分层架构简介

### 第四章 Abp的DDD分层（需特别强调.Shard和.Contracts分层）

1. 构建Abp分层项目
   1. 领域层
      1. 领域层概念简介、领域对象简介、abp定义的基类型简介
      2. 创建Book、Author等实体、编写领域逻辑、仓储
   2. 应用层
      1. 应用层概念简介、应用层对象简介、abp定义的基类型简介
      2. 设计用户用例，创建应用服务，AutoMapper
      3. 工作单元
   3. 展示层
      1. HttpApi接口定义
   4. 基础设置层
      1. ef core
      2. 仓储实现及注册
   5. 数据迁移及迁移控制台项目
   6. 运行项目
2. 总结

## 第三部分 框架

### 第五章 源码简介

1. 文件目录概述
2. framework
   1. src
   2. test
3. modules
   1. 示例module解决方案
      1. src
      2. test
4. templates

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

### 第八章 AOP

1. **AOP**概念简介
2. 动态代理，Interface介绍及框架中的AOP，**适配器模式**
3. 实现AOP
4. 自定义返回值（Filter？）

### 第九章 工作单元与数据库事务

1. 如何开启
   1. 约定
   2. 特性
   3. 手动控制
   4. 关闭
   5. 回调事件
2. 事务
   1. 开启事务
      1. 事务隔离级别
   2. 工作单元如何判断事务
      1. **AOP**
      2. **依赖倒置**
   3. 多DbContext
      1. Manager
      2. DatabaseApi和TransactionApi

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
