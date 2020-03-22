---
title: "Domain Fronting"
date: 2020-03-22T11:58:28+08:00
draft: flase
tags: [RedTeam]
categories: []
comment: true
---
红队基础建设之Domain Fronting
<!--more-->

# 红队基础建设之Domain Fronting

## 简介

Domain Fronting（域前缀），是一种隐藏真实C2服务器IP且同时能伪装为与高信誉域名通信的技术。

- 技术原理图

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321134624.png)

## 原理

**Domain Fronting的核心技术主要是利用了CDN**

### 举例

- nslookup

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321135734.png)

**两个网站使用的是同一个CDN**

- 请求klst.96jm.com (301状态)

```
< HTTP/1.1 301 Moved Permanently
< Server: Tengine
< Date: Sat, 21 Mar 2020 05:58:57 GMT
< Content-Type: text/html
< Content-Length: 278
< Connection: keep-alive
< Location: https://klst.96jm.com/
< Via: cache4.cn899[,0]
< Timing-Allow-Origin: *
< EagleId: 3cddc21815847703378007817e
<
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<h1>301 Moved Permanently</h1>
<p>The requested resource has been assigned a new permanent URI.</p>
<hr/>Powered by Tengine</body>
</html>
* Connection #0 to host klst.96jm.com left intact
```

- 请求 www.shhorse.com.cn （403状态）

```
< HTTP/1.1 403 Forbidden
< Server: Tengine
< Content-Type: text/html
< Content-Length: 597
< Connection: keep-alive
< Cache-Control: no-cache
< x-alicdn-da-ups-status: endOs,0,403
< Via: cache11.l2cm12-6[50,0], cache2.cn899[86,0]
< Timing-Allow-Origin: *
< EagleId: 3cddc21615847703796571626e
<
<html>
<head>
```

- 请求CDN

```
curl klst.96jm.com -H "www.shhorse.com.cn" -v
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321140832.png)

## 实践

### 配置CDN

- Tips

> 但由于某云有一个有趣的特点：当 CDN 配置中的源 IP 为自己云的服务器时，加速时会跳过对域名的检验，直接与配置中的域名绑定的源服务器IP进行通信。利用该特性，我们不需要真正去申请 CDN 时所填写的域名中配置解析相应的 CNAME 了。换言之，只要我们的C2服务器属于该云的服务器，那么我们就无需申请域名，只需要在申请 CDN 时随便填一个没有人绑定过的域名就好了，而且这个域名我们可以填成任何高信誉的域名，例如xxx.microsoft.com.

- 申请CDN

**马赛克处是你c2的IP**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321155847.png)

- 过几分钟查看状态

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321155741.png)

### 配置CobaltStrike

- 配置C2 profile

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321155655.png)

- WebLog查看请求

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321160517.png)

- 添加一个监听器

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321155142.png)

### Agent Generator

- 下载地址

```
https://codeload.github.com/mdsecactivebreach/CACTUSTORCH/zip/master
```

- 加载

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321152652.png)

- 生成

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321160640.png)

- 执行

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321160931.png)

- 查看通讯

```
Netstat -an
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200321161248.png)

**发现是与CDN进行通讯，隐藏了真实的c2**
## 参考

 ```
 https://www.anquanke.com/post/id/195011
 ```