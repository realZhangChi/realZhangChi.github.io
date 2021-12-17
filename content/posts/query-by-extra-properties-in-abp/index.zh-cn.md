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

Abp框架提供了实体扩展系统，允许在不对类的定义进行更改的情况下，向对象中添加额外的属性。默认地，额外属性是以json对象的形式存储在数据库表的`ExtraProperties`字段中，因此无法直接将额外属性作为查询条件。这篇文章将详细描述如何来根据额外属性进行查询。

# Entity Framework Core数据库映射

直接对ExtraProperties属性进行模糊查询，会存在性能问题，因此对于可能会作为存储查询条件的额外属性，需要将其映射为数据库表中单独的一列。