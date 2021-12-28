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

在上一篇教程中，分析了CatchE项目的业务逻辑，得到了核心域，并对`Issue`、`Answerer`聚合建模。这篇文章中将创建领域服务，进一步实现业务逻辑。

## 领域服务

领域服务处理业务规则。实现业务逻辑时，首先应考虑实体。如果实体无法**独立**实现业务，或者某个操作过程不属于实体的职责时，便应该将其放在领域服务中。

在CatchE中，提出一个`Issue`时，应选择一个`Answerer`回答问题。若不存在`Answerer`与选中的用户匹配，则根据选中用户进行创建。

考虑上述需求，通过构造函数创建`Issue`时，无法得知回答者是否存在，也无法创建回答者

