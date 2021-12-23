+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = ["Abp极简教程"]
date = 2021-12-23T04:12:10Z
description = "从控制台项目开始，轻松入门Abp框架。"
draft = true
tags = ["Tutorials", "Abp"]
title = "Abp极简教程-1 创建项目"

+++
Abp是一个基于.NET的开源应用程序框架，它遵循最佳实践和约定，根据DDD模式进行设计和开发，并提供了强大的基础设施和完整的架构。

Abp提供了项目启动模板，它依据DDD模式进行分层，并预先配置了常用的模块。启动模板中反映着领域驱动设计、软件工程中的最佳实践、设计模式等多种概念，其中任一项都值得单独讨论。对于初学者而言，启动模板中的大量知识如潮水般瞬间涌入脑海，造成知识过载，无法聚焦当前真正要学习的知识。

本系列教程，将从简单的.NET Web Api应用程序开始，搭建起基于Abp的Web应用程序框架，并逐步深入细节，旨在以一种缓和的学习曲线帮助初学者快速入门。教程中的每一篇文章，都将会针对特定的几个知识点进行阐述，来帮助读者实现关注点的聚焦，区分知识学习的先后顺序。本教程面向初学者或对Abp有了初步了解的人群。

## 创建应用

通过VS创建一个名为`BookStore`Web Api应用，选择ASP.NET Core Web Api项目模板，或通过CLI执行`dotnet new webapi -n BookStore`。运行后将访问至Swagger页面。

## 集成Abp

### 第一个模块

添加`Volo.Abp.Autofac`和`Volo.Abp.AspNetCore.Mvc` Nuget包引用至项目中，创建C#类文件命名为`BookStoreModule`更改代码如下：

    [DependsOn(
        typeof(AbpAutofacModule),
        typeof(AbpAspNetCoreMvcModule))]
    public class BookStoreModule : AbpModule
    {
        public override void ConfigureServices(ServiceConfigurationContext context)
        {
            ConfigureSwaggerServices(context);
        }
    
        public override void OnApplicationInitialization(ApplicationInitializationContext context)
        {
            var app = context.GetApplicationBuilder();
            var env = context.GetEnvironment();
    
            app.UseRouting();
            if (env.IsDevelopment())
            {
                app.UseSwagger();
                app.UseSwaggerUI();
            }
            app.UseConfiguredEndpoints();
        }
    
    
        private static void ConfigureSwaggerServices(ServiceConfigurationContext context)
        {
            context.Services.AddSwaggerGen();
        }
    }

Abp设计为模块化的应用程序框架，每一个模块都应定义一个继承自`AbpModule`的类，并以`Module`后缀作为类名。不同的模块间会存在依赖关系，模块的依赖关系通过`DependsOn`特性来定义。

在`ConfigureServices`方法中，可以将依赖项注册到依赖注入系统中。在Abp中，可以通过约定大于配置的方式进行依赖项注册，项目代码通常无需在这里手动注册。示例程序在在`ConfigureServices`方法中注册了Swagger相关服务。

在应用程序启动时，将会按照依赖顺序初始化所有的模块。初始化启动项模块时将会调用他的`OnApplicationInitialization`方法，通常在这个方法中会构建出中间件管道。示例程序配置了路由和终结点管道，并在开发环境中配置Swagger中间件。

### 配置Abp应用

Abp框架中定义了`IAbpApplication`应用。项目启动时应构建应用并运行。应用包含了启动模块及其依赖，构建应用时需要指定启动模块。