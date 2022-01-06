---
title: "Abp极简教程-6 Entity Framework与仓储实现"
date: 2022-01-06T11:22:17+08:00
draft: true
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "向CatchException项目中集成Entity Framework Core框架，并实现仓储。了解领域驱动设计中的基础设施层。"
tags: ["Tutorials", "Abp"]
categories: ["Abp极简教程"]
---

在前几篇教程中，对领域进行了建模，开发了领域层，针对应用程序用例开发了应用层，还创建了负责数据适配的HttpApi接口。到目前为止，项目还无法运行起来，因为没有为仓储提供实现。这篇教程中将集成Entity Framework Core并实现仓储。

## 集成Entity Framework Core

### 模块化

新建项目`CatchException.EntityFrameworkCore`，添加Nuget包引用`Volo.Abp.EntityFrameworkCore.MySQL`，添加项目引用`CatchException.Domain`，创建Abp模块`CatchExceptionEntityFrameworkCoreModule`。

```cs
[DependsOn(
    typeof(AbpEntityFrameworkCoreMySQLModule),
    typeof(CatchExceptionDomainModule))]
public class CatchExceptionEntityFrameworkCoreModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        Configure<AbpDbContextOptions>(options =>
        {
            options.UseMySQL();
        });
    }
}
```

### 创建DbContext

创建`CatchExceptionDbContext`并配置实体数据模型。在配置实体模型时，需要通过`ConfigureByConvention`扩展方法来处理审计字段等。只需将聚合根的`DbSet`作为属性添加到`CatchExceptionDbContext`中。

```cs
public class CatchExceptionDbContext
    : AbpDbContext<CatchExceptionDbContext>
{
    public DbSet<Issue> Issues { get; set; }

    public DbSet<Answerer> Answerers { get; set; }

    public CatchExceptionDbContext(DbContextOptions<CatchExceptionDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<Issue>(b =>
        {
            b.ToTable("Issues");

            b.ConfigureByConvention();
            b.Property(p => p.Title).HasMaxLength(128);
            b.Property(p => p.Description).HasMaxLength(2048);
        });

        modelBuilder.Entity<Answerer>(b =>
        {
            b.ToTable("Answerers");

            b.ConfigureByConvention();
        });
    }
}
```

### 数据迁移

创建设计时`DbContext`工厂，以供执行数据迁移命令时使用。

```cs
public class CatchExceptionDbContextFactory : IDesignTimeDbContextFactory<CatchExceptionDbContext>
{
    public CatchExceptionDbContext CreateDbContext(string[] args)
    {
        var configuration = BuildConfiguration();

        var builder = new DbContextOptionsBuilder<CatchExceptionDbContext>()
            .UseMySql(configuration.GetConnectionString("Default"), MySqlServerVersion.LatestSupportedServerVersion);

        return new CatchExceptionDbContext(builder.Options);
    }

    private static IConfigurationRoot BuildConfiguration()
    {
        var builder = new ConfigurationBuilder()
            .SetBasePath(Path.Combine(Directory.GetCurrentDirectory(), "../CatchException.HttpApi.Host/"))
            .AddJsonFile("appsettings.json", optional: false)
            .;

        return builder.Build();
    }
}
```

在CatchException.HttpApi.Host项目中配置连接字符串。

```json
{
  "ConnectionStrings": {
    "Default": "Server=127.0.0.1;Database=CatchException;User=root;Password=yourStrong(!)Password"
  }
}
```

添加Nuget包引用`Microsoft.EntityFrameworkCore.Tools`，并在`CatchException.EntityFrameworkCore`项目目录下生成迁移文件。

```sh
dotnet ef migrations add Initial
```

此时在`CatchException.EntityFrameworkCore`项目中已生成迁移文件，接下来即可将将数据结构更新至数据库。

```sh
dotnet ef database update
```

## 注册仓储

在`CatchExceptionEntityFrameworkCoreModule`将`CatchExceptionDbContext`注册到依赖注入系统容中，并创建默认的泛型仓储。

```cs
context.Services.AddAbpDbContext<CatchExceptionDbContext>(options =>
{
    options.AddDefaultRepositories(includeAllEntities: false);
});
```

