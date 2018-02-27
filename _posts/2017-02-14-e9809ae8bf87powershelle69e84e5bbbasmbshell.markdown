---
author: heh3.org
comments: true
date: 2017-02-14 09:26:12+00:00
layout: post
link: http://heh3.org/?p=8
slug: '%e9%80%9a%e8%bf%87powershell%e6%9e%84%e5%bb%basmbshell'
title: 通过powershell构建SMBshell
wordpress_id: 8
categories:
- ps
- tools
tags:
- powershell
- smbshell
---

## 通过powershell构建SMBshell


[https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Invoke-SMBShell.ps1](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Invoke-SMBShell.ps1)

建立连接之前需要使用net use 建立一个连接smb连接
(eg: net use \server\share)


### 服务端：


先导入模块，import module Invoke-SMBShell

`C:\PS> Invoke-SMBShell`

![](http://i.imgur.com/njsGTkp.png)


### 客户端：


`C:\PS> Invoke-SMBShell -Client -Server REDRUM-DC -AESKey dFqeSRa5IVD7Daby -Pipe tapsrv.5604.w6YjHCgSOVpoXOZF`

![](http://i.imgur.com/r53EHXv.png)
