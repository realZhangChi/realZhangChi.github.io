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

Abp框架提供了实体扩展系统，允许在不对类的定义进行更改的情况下，向对象中添加额外的属性。额外属性是以json对象的形式存储在数据库表的`ExtraProperties`字段中，因此无法直接将额外属性作为查询条件。这篇文章将详细描述如何来根据额外属性进行查询。

# 通过EF.Functions

EF Core通过定义`EF.Functions`扩展方法来调用某些数据库函数，`JsonContains`就是其中之一。`JsonContains`可判断值为Json对象格式的某一列中，是否包含指定Json对象。在Abp中，当需要以额外属性作为条件进行查询时，可在`EntityFrameworkCore`的仓储中调用`JsonContains`来实现。