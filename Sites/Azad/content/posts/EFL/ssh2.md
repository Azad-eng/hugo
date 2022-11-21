---
title: "如何使用 SSH 控制连接 Windows 服务器？"
description: 
tags: [ "SSH", "实用工具" ]
categories:
  - "工作总结"
date: 2022-06-01T15:03:45+08:00
lastmod: 2022-06-01T15:03:45+08:00
draft: false
---
### 在windows上安装SSH服务器
[windows自带SSH服务器：应用和功能-管理可选功能] 

### 用管理员身份启动PowerShell自动化部署OpenSSH 服务器

* **确保OpenSSH 可用于安装：**
``` powershell
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
```


* **安装：**
``` powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```


* **初始化配置**

1. 开启 SSHD 服务：
``` powershell
Start-Service sshd
```


2. 设置服务的自启动：
``` powershell
Set-Service -Name sshd -StartupType 'Automatic'
```


3. 确认防火墙：
``` powershell
Get-NetFirewallRule -Name *ssh*
```
