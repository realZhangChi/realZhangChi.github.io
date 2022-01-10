---
title: Abp极简教程6 Entity Framework与仓储实现
date: 2022-01-06T11:22:17.000+08:00
author: 张驰
authorLink: https://github.com/realZhangChi
description: 向CatchException项目中集成Entity Framework Core框架，并实现仓储。了解领域驱动设计中的基础设施层。
tags:
- Tutorials
- Abp
categories:
- Abp极简教程
draft: true

---
在前几篇教程中，对领域进行了建模，开发了领域层，针对应用程序用例开发了应用层，还创建了负责数据适配的HttpApi接口。到目前为止，项目还无法运行起来，因为没有为仓储提供实现。这篇教程中将集成Entity Framework Core并实现仓储。

## 集成Entity Framework Core

数据库作为基础设施，为实体仓储的持久化提供了支持。在CatchException项目中，将创建Entity Framework Core基础设施项目，负责集成数据库，并实现领域对象和数据库之间的数据适配。

{{< admonition tip "整洁架构">}}
在整洁架构中，数据库位于最外层的“框架与驱动程序”中，而Entity Framework Core基础设施项目则位于“接口与适配器层”，它负责将数据从便于项目开发的C#类的格式，转换为便于持久化到数据库的格式。本教程最后一篇将会对分层架构进行简单的阐述。
{{< /admonition >}}

### Entity Framework Core模块

新建项目CatchException.EntityFrameworkCore，添加以下Nuget包引用集成Entity Framework Core。

- Microsoft.EntityFrameworkCore
- Microsoft.EntityFrameworkCore.Relational
- Microsoft.EntityFrameworkCore.Tools

创建Abp模块`CatchExceptionEntityFrameworkCoreModule`，添加Abp的Nuget包`Volo.Abp.EntityFrameworkCore.MySQL`引用及`AbpEntityFrameworkCoreMySQLModule`模块依赖，添加项目引用`CatchException.Domain`及`CatchExceptionDomainModule`模块依赖，因为项目依赖Identity通用子域，因此还需添加`Volo.Abp.Identity.EntityFrameworkCore`包引用及`AbpIdentityEntityFrameworkCoreModule`模块依赖。

```cs
[DependsOn(
    typeof(AbpEntityFrameworkCoreMySQLModule),
    typeof(AbpIdentityEntityFrameworkCoreModule),
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

创建`CatchExceptionDbContext`并将聚合根的`DbSet`作为属性。在配置实体模型时，需要通过`ConfigureByConvention`扩展方法配置审计字段。`ConfigureIdentity`将配置Identity模块的实体。

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

        modelBuilder.ConfigureIdentity();

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

在`CatchException.EntityFrameworkCore`项目目录下执行命令`dotnet ef migrations add Initial`生成迁移文件，并执行`dotnet ef database update`更新数据库。

## 注册仓储

`AddDefaultRepositories`扩展方法将会为`CatchExceptionDbContext`中的聚合根`DbSet`注册默认的泛型仓储。默认地，只会为聚合根注册默认实现的反省仓储，若要为普通实体注册，将`includeAllEntities`参数改为`true`并将实体`DbSet`也添加到`CatchExceptionDbContext`中。

因为`CatchExceptionEntityFrameworkCoreModule`依赖了`AbpIdentityEntityFrameworkCoreModule`模块，在应用程序初始化时调用`CatchExceptionEntityFrameworkCoreModule`的`ConfigureServices`方法之前，会先调用`AbpIdentityEntityFrameworkCoreModule`模块的`ConfigureServices`方法来为其模块内的聚合根注册泛型仓储。

```cs
[DependsOn(
    typeof(AbpEntityFrameworkCoreMySQLModule),
    typeof(AbpIdentityEntityFrameworkCoreModule),
    typeof(CatchExceptionDomainModule))]
public class CatchExceptionEntityFrameworkCoreModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddAbpDbContext<CatchExceptionDbContext>(options =>
        {
            options.AddDefaultRepositories(includeAllEntities: false);
        });

        Configure<AbpDbContextOptions>(options =>
        {
            options.UseMySQL();
        });
    }
}
```

## 整合起来

基于依赖倒置的原则，CatchException.Domain不会依赖CatchException.EntityFrameworkCore具体实现，而是具体实现将依赖于Domain中的仓储接口。通过项目分层，将依赖项倒置了，那么还需要通过HttpApi.Host启动项目来将其整合起来。

{{< admonition note "依赖倒置">}}
依赖于抽象，而不是具体实现。
{{< /admonition >}}

将CatchException.EntityFrameworkCore项目引用添加至CatchException.HttpApi.Host项目中，并将`CatchExceptionEntityFrameworkCoreModule`模块添加到`CatchExceptionHttpApiHostModule`模块依赖项中。通过Host启动项目，将所有模块整合到一起，此时运行程序并调用接口，即可实现`Issue`的创建。

## 总结

在这篇教程中，集成了Entity Framework Core框架，并通过EF Core实现了仓储，此外，还了解到了整洁架构中的“框架与驱动程序”、“接口与适配器”。下一篇教程中将会创建HttpApi.Client客户端代理项目。