---
title: "Security Support Provider"
date: 2019-08-09T17:04:05+08:00
lastmod: 2019-08-09T17:04:05+08:00
draft: false
tags: [域渗透]
categories: [内网渗透]
comment: true
---
Security Support Provider
<!--more-->
# Security Support Provider
## 简介
**来源：**
[Security Support Provider](https://wooyun.js.org/drops/%E5%9F%9F%E6%B8%97%E9%80%8F%E2%80%94%E2%80%94Security%20Support%20Provider.html)

>SSP：
Security Support Provider，直译为安全支持提供者，又名Security Package.简单的理解为SSP就是一个DLL，用来实现身份认证

>SSPI：
Security Support Provider Interface，直译为安全支持提供程序接口，是Windows系统在执行认证操作所使用的API。简单的理解为SSPI是SSP的API接口

>LSA：
Local Security Authority，用于身份认证，常见进程为lsass.exe.特别的地方在于LSA是可扩展的，在系统启动的时候SSP会被加载到进程lsass.exe中.
这相当于我们可以自定义一个dll，在系统启动的时候被加载到进程lsass.exe！

## 测试
### 使用注册表
- 添加SSP
![](https://pic.superbed.cn/item/5d4d38d4451253d1780a02d3.jpg)

- 修改注册表位置
![](https://pic.superbed.cn/item/5d4d38d4451253d1780a02d3.jpg)

- 添加mimilib.dll
![](https://pic.superbed.cn/item/5d4d39f7451253d1780a11a2.jpg)

- 重启系统验证
![](https://pic.superbed.cn/item/5d4d3bd8451253d1780a2484.jpg)
### 进程注入（重启失效）
![](https://pic1.superbed.cn/item/5d4d3c46451253d1780a2868.jpg)
