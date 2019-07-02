---
layout: post
title: "使用NLog向数据库中写日志"
subtitle: 'NLog Database Target'
author: "Chi"
date: 2019-06-30 22:35
header-style: text
catalog: false
tags:
  - NLog
---

这篇文章介绍了如何使用NLog通过ADO.NET来向数据库中写日志，数据库操作不会作为事务来执行。

> 平台支持：**限制**(不支持Silverlight和Xamarin Android/iOS)。

对于.NET Core 用户，如果SQL Server数据库，需要在项目中添加Nuget包`System.Data.SqlClient`。

## 进行配置

``` xml
<targets>
  <target xsi:type="Database"
          name="String"
          dbProvider="String"
          connectionString="Layout"
          connectionStringName="String"
          keepConnection="Boolean"
          dbDatabase="Layout"
          dbUserName="Layout"
          dbPassword="Layout"
          dbHost="Layout"
          commandType="Enum"
          commandText="Layout"
          installConnectionString="Layout">
    <install-command commandType="Enum"
                     connectionString="Layout"
                     ignoreFailures="Boolean"
                     text="Layout"/><!-- repeated -->
    <uninstall-command commandType="Enum"
                       connectionString="Layout"
                       ignoreFailures="Boolean"
                       text="Layout"/><!-- repeated -->
    <parameter name="String"
              layout="Layout"
              precision="Byte"
              scale="Byte"
              size="Integer"
              dbType="DbType"
              format="string"
              parameterType="Type" /> <!-- repeated -->
  </target>
</targets>
```

## 参数

### 通用选项

- name -目标名(待进一步解释)

### 数据库连接相关的配置

- dbProvider - 数据库连接提供器名。sqlserver默认情况下会识别以下值：

sqlserver, mssql, microsoft or msde - 识别为System.Data.SqlClient数据提供器。
odbc - ODBC数据提供器(不支持.NET Core)
oledb - OLEDB数据提供器(不支持.NET Core)

注意.NET Core应该为DbProvider安装Nuget-package（例如System.Data.SqlClient），而且使用提供者连接类型的完全限定名称（实现IDbConnection的类）。[了解更多](https://github.com/NLog/NLog/wiki/Database-target#dbprovider-examples)

- connectionString - 连接字符串。

待更新...
