+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = [".NET"]
date = 2021-12-22T06:22:01Z
description = "如何在生命周期为Singleton的服务中，解析生命周期为Scoped或Transient的依赖项。 "
tags = ["dependency injection", ".NET"]
title = "在单例服务中解析依赖项"

+++
在依赖注入系统中，依赖项的生命周期通常分为瞬时的（Transient）、作用域的（Scoped）、单例的（Singleton）三种。单例生命周期的服务通常会在首次调用时创建，后续每此调用都会使用同一实例。

单例服务若依赖其他生命周期为瞬时或作用域的服务时，无法通过构造函数注入依赖项。构造函数只会在创建实例时调用一次，若将依赖项通过构造函数注入并赋值给单例服务的本地成员，依赖项的生命周期结束后销毁后，指向依赖项的本地成员将会指向空引用，且永远不会再次被赋值（只在调用构造函数时赋值）。运行时会抛出异常`Cannot consume scoped service 'XXX' from singleton 'XXX'.`。

## 解决方案

在依赖瞬时生命周期或作用域生命周期依赖项的单例服务中，不直接通过构造函数注入依赖项，而是注入`IServiceScopeFactory`，在需要用到依赖项的方法中，通过`IServiceScopeFactory`创建作用域并解析依赖项。

    public class MySingletonService : IMySingletonService
    {
        private readonly IServiceScopeFactory _scopeFactory;
    
        public MySingletonService(IServiceScopeFactory scopeFactory)
        {
            _scopeFactory = scopeFactory;
        }
        
        public void Scoped()
        {
            using var scope = _scopeFactory.CreateScope();
            var ctx = scope.ServiceProvider.GetRequiredService<MyDbContext>();
        }
    }
    