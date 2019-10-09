---
title: "Skeleton Key"
date: 2019-08-15T16:57:41+08:00
lastmod: 2019-08-15T16:57:41+08:00
draft: false
tags: [域渗透]
categories: [内网渗透]
comment: true
---
Skeleton Key
<!--more-->
# Skeleton Key
>Skeleton Key被安装在64位的域控服务器上
支持Windows Server2003—Windows Server2012 R2
能够让所有域用户使用同一个万能密码进行登录
现有的所有域用户使用原密码仍能继续登录
重启后失效
Mimikatz(Version 2.0 alpha,20150107)支持 Skeleton Key

## 域内主机使用正确密码登录域控
```
net use \\OWA2010SP3.0day.org admin!@#45 /user:jack
dir \\OWA2010SP3.0day.org\c$
```
