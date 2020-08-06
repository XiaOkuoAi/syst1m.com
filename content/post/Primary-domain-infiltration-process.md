---
title: "Primary Domain Infiltration Process"
date: 2020-08-06T22:07:02+08:00
draft: flase
tags: []
categories: [域渗透]
comment: true
---

一次平淡无奇的域渗透
<!--more-->

# 一次平淡无奇的域渗透

**这是一次很普通的渗透过程，好兄弟撕夜于网站上发现了SQL注入，为一处登陆处的注入，注入点为用户名或密码。说着我们复制了post包放入sqlmap目录下，直接来一发 -r**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806215003.png)

- 注入
 
```
POST /XXXXXX/XXX/Login.asp HTTP/1.1Host: target.comUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:78.0) Gecko/20100101 Firefox/78.0Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Content-Type: application/x-www-form-urlencodedContent-Length: 29Origin: http://target.comConnection: closeName=admin*&PassWord=asdasd
```

**我们发现支持堆叠注入，直接使用cs生成pscommand于os-shell中执行，上线到cs上便于操作。**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806213001.png)

- Cobalt Strike

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806170834.png)

**我们进入命令行发现机器为域内机器，为普通用户，所以首先第一步想要把权限提升至高权限用户**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806171436.png)

**查看系统补丁情况，到补丁缺少查询网站查询，发现缺少ms16-075，到github中下载对应的提取插件，使用插件进行权限提升，成功将权限提升至system权限**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806213748.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806171907.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806172346.png)

**查看了域内机器情况，发现有名为AD1、AD2、AD3的一台主控，两台辅控机器**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806213951.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806173031.png)

**在当前已经提升了权限的高权限下dump hash，获取到部分用户账户，这时候一个惊喜发生了，就是通过dump hash发现了域管的账号情况，这不是美滋滋吗**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806173723.png)

**但是首先我们将shell注入到了其他进程，万一一下掉了呢对吧**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806173827.png)

**接下来进行pth，使用已经获取的域管账号hash对其他机器进行移动，首先就是AD1的域控**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806174332.png)

**成功获取到了AD1域控的SYSTEM权限**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806174556.png)

**于AD1上进行dump hash**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806174814.png)

**将域管账号对其他机器进行批量pth，梭哈就完事了**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806221913.png)
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806215338.png)

**最后获得了三台AD，mail，file一些权限，然后就去看电视了**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200806194313.png)