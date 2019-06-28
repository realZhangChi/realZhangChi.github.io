---
layout: post
title: "Entity Framework和事务"
subtitle: 'Hello World, Hello My New Life!'
author: "Chi"
date: 2019-06-28 09:00
header-style: text
catalog: true
tags:
  - EF
  - 数据库
---

## 数据库事务

数据库事务（简称：事务）是数据库管理系统执行过程中的一个逻辑单位，由一个有限的数据库操作序列构成。事务允许以原子方式处理多个数据库操作。 如果已提交事务，则所有操作都会成功应用到数据库。 如果已回滚事务，则所有操作都不会应用到数据库。

数据库事务通常包含了一个序列的对数据库的读/写操作。包含有以下两个目的：

1. 为数据库操作序列提供了一个从失败中恢复到正常状态的方法，同时提供了数据库即使在异常状态下仍能保持一致性的方法。
2. 当多个应用程序在并发访问数据库时，可以在这些应用程序之间提供一个隔离方法，以防止彼此的操作互相干扰。

当事务被提交给了数据库管理系统（DBMS），则DBMS需要确保该事务中的所有操作都成功完成且其结果被永久保存在数据库中，如果事务中有的操作没有成功完成，则事务中的所有操作都需要回滚，回到事务执行前的状态；同时，该事务对数据库或者其他事务的执行无影响，所有的事务都好像在独立的运行。

并非任意的对数据库的操作序列都是数据库事务。数据库事务拥有以下四个特性，习惯上被称之为ACID特性。

- 原子性（Atomicity）：事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行。
- 一致性（Consistency）：事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束。
- 隔离性（Isolation）：多个事务并发执行时，一个事务的执行不应影响其他事务的执行。
- 持久性（Durability）：已被提交的事务对数据库的修改应该永久保存在数据库中。

## 在EF中使用事务

默认情况下，如果数据库提供程序支持事务，则会在事务中应用对`SaveChanges()`的单一调用中的所有更改。 如果其中有任何更改失败，则会回滚事务且所有更改都不会应用到数据库。 这意味着，`SaveChanges()` 可保证完全成功，或在出现错误时不修改数据库。

对于大多数应用程序，默认行为已足够。 如果应用程序要求被视为有必要，则应该仅手动控制事务。
可以使用 DbContext.Database API 开始、提交和回滚事务。

``` C#
using (var context = new BloggingContext())
{
    using (var transaction = context.Database.BeginTransaction())
    {
        try
        {
            context.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            context.SaveChanges();

            context.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            context.SaveChanges();

            var blogs = context.Blogs
                .OrderBy(b => b.Url)
                .ToList();

            // Commit transaction if all commands succeed, transaction will auto-rollback
            // when disposed if either commands fails
            transaction.Commit();
        }
        catch (Exception)
        {
            // TODO: Handle failure
        }
    }
}
```

## 参考

> [数据库事务-维基百科](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1)
>
> [事务-EF Core | Microsoft Docs](https://docs.microsoft.com/zh-cn/ef/core/saving/transactions)

