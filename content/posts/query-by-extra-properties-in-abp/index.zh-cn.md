+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = ["abp"]
date = 2021-12-17T08:16:35Z
description = ""
draft = true
tags = ["abp"]
title = "query-by-extra-properties-in-abp"

+++
# Intro

Abp框架提供了实体扩展系统，允许在不对类的定义进行更改的情况下，向对象中添加额外的属性。默认地，额外属性是以json对象的形式存储在数据库表的`ExtraProperties`字段中，因此无法直接将额外属性作为查询条件。对于额外属性，Abp支持将其通过Entity Framework Core映射为数据库表的单独字段，因此我们可以利用数据库映射来实现根据额外属性进行查询。

# 数据库映射

将额外属性映射为数据库表字段非常容易。

通过Abp启动模板创建的解决方案中，预先生成了处理数据库映射的`*EfCoreEntityExtensionMappings`类，它位于*.EntityFrameworkCore项目中。在项目启动时，将会执行其中的`Configure`方法，通过`OneTimeRunner`执行一次操作。

在`OneTimeRunner.Run()`方法的Action参数中，通过`ObjectExtensionManager`来处理额外属性到数据库表字段的映射。

    ObjectExtensionManager.Instance
        .AddOrUpdateProperty<IdentityUser, int>(
            "Gender",
            options =>
            {
                options.MapEfCore((_, p) => p.HasMaxLength(8));
            }
        );

通过以上代码即可完成额外属性到表字段的映射。添加数据迁移脚本并运行*.DbMigrator更新数据库接口，可以看到表中多出一个名为`Gender`的字段。

# 查询