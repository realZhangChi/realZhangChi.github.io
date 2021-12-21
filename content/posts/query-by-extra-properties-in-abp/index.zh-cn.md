+++
author = "张驰"
authorLink = "https://github.com/realZhangChi"
categories = ["abp"]
date = 2021-12-17T08:16:35Z
description = ""
tags = ["abp"]
title = "在Abp中根据ExtraProperties进行查询"

+++
# Intro

Abp框架提供了实体扩展系统，允许在不对类的定义进行更改的情况下，向对象中添加额外的属性。默认地，额外属性是以json对象的形式存储在数据库表的`ExtraProperties`字段中，因此无法直接将额外属性作为查询条件。对于额外属性，Abp支持将其通过Entity Framework Core映射为数据库表的单独字段，因此我们可以利用数据库映射来实现根据额外属性进行查询。

# 数据库映射

将额外属性映射为数据库表字段非常容易。

通过Abp启动模板创建的解决方案中，预先生成了处理数据库映射的`*EfCoreEntityExtensionMappings`类，它位于*.EntityFrameworkCore项目中。在项目启动时，将会执行其中的`Configure`方法，通过`OneTimeRunner`执行一次操作。

在`OneTimeRunner.Run()`方法的Action参数中，通过`ObjectExtensionManager`来处理额外属性到数据库表字段的映射。

    ObjectExtensionManager.Instance
        .AddOrUpdateProperty<IdentityUser, string>(
            "Gender",
            options =>
            {
                options.MapEfCore((b, p) =>
                {
                    b.HasIndex("Gender");
                    p.IsRequired().HasDefaultValue(string.Empty);
                    p.HasMaxLength(8);
                });
            }
        );

在`AddOrUpdateProperty`方法中还可以设置表字段长度等，也可设置表的属性如索引。

添加数据迁移脚本并运行*.DbMigrator更新数据库接口，可以看到表中多出一个名为`Gender`的字段。

# 查询

在*.EntityFramework.Core项目中创建仓储，并创建查询方法。

    public async Task<IdentityUser> GetUserByGenderAsync(string gender)
    {
        return await (await GetDbSetAsync())
            .FromSqlRaw($"select * from AbpUsers where Gender == '{gender}'")
            .FirstOrDefaultAsync();
    }

调用方法`GetUserByGenderAsync`并传入`gender`参数即可根据`Gender`进行查询。

# Summary

在这篇文章中，描述了如何对额外属性进行数据库映射，以及将额外属性作为查询条件检索数据。值得注意的是，将额外属性作为查询条件并不是最佳实践，如果可能的话应当尽量避免。此外，如需将拥有额外属性的`Entity`通过AutoMapper映射为`Dto`，不要忘记对`Dto`进行扩展并配置`AutoMapperProfile`。