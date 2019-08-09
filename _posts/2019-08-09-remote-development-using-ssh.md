---
layout: post
title: "使用VS Code进行远程开发"
subtitle: 'Remote Development using SSH'
author: "Chi"
date: 2019-08-09 17:31
header-style: text
catalog: false
tags:
  - VS Code
---

## 使用VS Code进行远程开发

VS Code已经有了远程开发的功能。在此之前我曾经遇见过一个场景，在利用Docker来进行ASP .NET Core开发的时候，本机的Docker Desktop经常会出现莫名其妙的问题，那时我就在寻找一种连接远程Linux服务器进行开发的方案。VS Code Remote开发的功能已经发布有一段时间了，今天终于有空进行一次尝试。

## 本机Windows配置

VS Code Remote是基于SSH来实现的，那么我们的Windows本机则需要安装配置SSH。

1. 以管理员模式打开PowerShell，运行命令`Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'`：

```PowerShell
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'

# This should return the following output:

Name  : OpenSSH.Client~~~~0.0.1.0
State : NotPresent
Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent
```

2. 安装SSH客户端和服务端：

```PowerShell
# 安装 OpenSSH 客户端
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# 安装 OpenSSH 服务端
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Both of these should return the following output:

Path          :
Online        : True
RestartNeeded : False
```


3. 初始化SSH服务配置：

```PowerShell
Start-Service sshd
# OPTIONAL but recommended:
Set-Service -Name sshd -StartupType 'Automatic'
# Confirm the Firewall rule is configured. It should be created automatically by setup.
Get-NetFirewallRule -Name *ssh*
# There should be a firewall rule named "OpenSSH-Server-In-TCP", which should be enabled
```

我们可以连接来测试一下：

```PowerShell
Ssh 你的服务器用户名@你的服务器地址

The authenticity of host 'servername (10.00.00.001)' can't be established.
ECDSA key fingerprint is SHA256:(<a large string>).
Are you sure you want to continue connecting (yes/no)?
```

4. 配置SSH Key
 VS Code是使用SSH配置文件和基于SSH Key的验证来连接服务器的。
 在PowerShell中输入命令`ssh-keygen -t rsa -b 4096`：

```PowerShell
ssh-keygen -t rsa -b 4096
```

此命令将在`%USERPROFILE%\.ssh\`文件夹下生成`id_rsa.pub`文件。
然后将本地Key（`id_rsa.pub`文件）的内容添加到SSH主机上相应的`authorized_keys`文件中。

```PowerShell
SET REMOTEHOST=你的服务器用户名@你的服务器地址

scp %USERPROFILE%\.ssh\id_rsa.pub %REMOTEHOST%:~/tmp.pub

ssh %REMOTEHOST% "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat ~/tmp.pub >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && rm -f ~/tmp.pub"
```

如果执行以上三条命令时出错的话，可以尝试将`%REMOTEHOST%`替换为`你的服务器用户名@你的服务器地址`，将`%USERPROFILE%\`替换为用户文件夹路径。

## VS Code配置

1. 在VS Code中安装扩展`Remote Development`

![Remote Development](..\img\in-post\2019-08-09-remote-development\2019-08-09-remote-development.png)

2. 在VS Code中按下<kbd>F1</kbd>键，输入`Remote-SSH: Connect to Host`，可以按照示例输入`服务器用户名@服务器地址`，也可以选择生成一个SSH 配置文件，在相应位置输入用户名和地址即可：

![ssh_user](..\img\in-post\2019-08-09-remote-development\ssh-user@box.png)

至此配置完成，可以连接到服务器进行远程开发了。关于如何利用Docker和 ASP .NET Core来进行开发，稍后我会更新。

> [Remote Development using SSH](https://code.visualstudio.com/docs/remote/ssh#_getting-started)
> [Install an OpenSSH compatible SSH client ](https://code.visualstudio.com/docs/remote/troubleshooting#_installing-a-supported-ssh-client)
> [Configuring key based authentication](https://code.visualstudio.com/docs/remote/troubleshooting#_configuring-key-based-authentication)
