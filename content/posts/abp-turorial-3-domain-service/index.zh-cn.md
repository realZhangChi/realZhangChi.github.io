---
title: "Abp极简教程-3 领域服务"
date: 2021-12-28T09:41:38+08:00
draft: true
author: "张驰"
authorLink: "https://github.com/realZhangChi"
description: "在CatchE项目中引入领域服务，来处理业务逻辑。"
tags: ["Tutorials", "Abp"]
categories: ["Abp极简教程"]
---

在上一篇教程中，我们对CatchE项目中的“问题”建模，得到了聚合根`Issue`、实体`Answer`。这篇文章中将创建领域服务，进一步实现业务逻辑。

## 领域服务

领域服务处理业务规则，是通用语言在代码中的体现。将业务逻辑实现为代码时，首先应考虑实体，如果实体无法**独立**实现，或者某个操作过程不属于实体的职责时，便应该将其放在领域服务中。在上一篇文章中已经实现了`Issue`和`Answer`的业务逻辑，接下来将创建一个领域服务，来实现实体之外的操作。

