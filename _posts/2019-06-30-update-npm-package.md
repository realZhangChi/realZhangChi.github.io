---
layout: post
title: "如何更新npm包"
subtitle: 'How to update npm package'
author: "Chi"
date: 2019-06-30 21:14
header-style: text
catalog: false
tags:
  - npm
  - package.json
---

在我搭建这个博客系统的时候，Github向我发送了多个依赖包版本低的警告，作为一个强迫症患者是无法容忍这件事情的，所以需要找到一个简便的方法来全局更新所有过期的包。

## 正文

发现参考的这个博客系统使用的是npm来进行包的管理，而在此之前我是没有在这台电脑的使用过npm的，所以需要先从[Nodejs官网](https://nodejs.org/en/)下载并安装Node.js和npm。需要注意的是，我使用的是VS Code进行开发，那么安装完Node.js后是需要重启VS Code的，否则可能仍然会报出`无法将“npm”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。请检查名称的拼写，如果包括路径，请确保路径正确，然后再试一次`的错误。

安装完成之后，就可以在项目中检查并更新依赖包了

1. `npm-check-updates`可以使用所有依赖项的最新版本自动调整package.json：

``` shell
npm install -g npm-check-updates
```

`npm-check-updates`只会更改我们的package.json文件。

2. 使用`ncu`查看当前目录下项目中存在的依赖项有哪些更新(这一步骤可以跳过，直接执行第三步即可。)：

``` shell
$ ncu
[====================] 5/5 100%

 express           4.12.x  →   4.13.x
 multer            ^0.1.8  →   ^1.0.1
 react-bootstrap  ^0.22.6  →  ^0.24.0
 react-a11y        ^0.1.1  →   ^0.2.6
 webpack          ~1.9.10  →  ~1.10.5

Run ncu -u to upgrade package.json
```

3. 使用`ncu -u`更新项目的package.json文件：

``` shell
$ ncu -u
Upgrading package.json
[====================] 1/1 100%

 express           4.12.x  →   4.13.x
```

4. 运行`npm install`来安装新版本的包

在第一步的时候我们提到过，`npm-check-updates`只会更改我们的package.json文件，那么运行`npm install`则会进一步更新我们package-lock.json文件中记录的依赖项包版本。

``` shell
Run npm install to install new versions.

$ npm install      # update installed packages and package-lock.json
```

执行到此即可全局更新所有依赖的包，包括package.json和package-lock.json文件。

## 参考

> [How do I update each dependency in package.json to the latest version?](https://stackoverflow.com/questions/16073603/how-do-i-update-each-dependency-in-package-json-to-the-latest-version)
> [npm-check-updates](https://www.npmjs.com/package/npm-check-updates)
