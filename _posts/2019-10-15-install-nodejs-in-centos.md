---
layout: post
title: "在 CentOS 7 中安装 Node.js 10.X LTS 最新版"
subtitle: 'Install Node.js 10.X LTS latest version in CentOS 7'
author: "Chi"
date: 2019-10-15 16:19
header-style: text
catalog: false
tags:
  - CentOS 7
  - Node.js
---

在CentOS 7中，Node.js的yum源默认不是最新发行版。那么我们可以采用二进制发行版来安装最新Node.js LTS。

执行命令：

``` bash
curl -sL https://rpm.nodesource.com/setup_10.x | bash -
```

这个命令将为我们配置Node.js NPM存储库。

配置了存储库后，清理yum源缓存并选择最快的源重新生成缓存：

``` bash
sudo yum clean all && sudo yum makecache fast
```

安装编译环境：

``` bash
sudo yum install -y gcc-c++ make
```

安装Node.js:

``` bash
sudo yum install -y nodejs
```

查看Node.js版本：

``` bash
node -v
```

如果输出的版本不是最新的Node.js版本的话，那么执行命令`yum remove nodejs`将已安装的Node.js移除，然后重新安装即可。

> [Install Node.js 10 LTS on CentOS 7 / Fedora 29 / Fedora 28](https://computingforgeeks.com/installing-node-js-10-lts-on-centos-7-fedora-29-fedora-28/)
