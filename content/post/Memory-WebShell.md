---
title: "Memory WebShell"
date: 2020-09-15T16:54:51+08:00
draft: false
tags: [内存马]
categories: [webshell]
comment: true
---
[防守视角] tomcat内存马的多种查杀方式
<!--more-->

# [防守视角] tomcat内存马的多种查杀方式

- 环境搭建

**我在WINDOWS7虚拟机下搭建的Tomcat，搭建教程网上都有，点击startup.bat启动环境**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200914221121.png)

- 设置内存马

**这里使用了哥斯拉的内存马**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200914234501.png)

## VisualVM（远程调试）


- 设置jstatd.all.policy 文件

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200914221527.png)

- 启动jstatd

```
jstatd.exe -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.hostname=serverip
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200914225845.png)

- 设置JVM Connection

**修改 catalina.sh文件(LINUX)**

```
JAVA_OPTS="-Djava.rmi.server.hostname=服务器的ip
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=jmx使用的端口
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false $JAVA_OPTS"
export JAVA_OPTS
```

**修改catalina.bat文件(WINDOWS)**

```
set JAVA_OPTS=-Djava.rmi.server.hostname=192.168.67.115 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8888 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false
```

- 下载VisualVM

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200914230218.png)

- MBeans安装插件

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200914230540.png)


- 连接远程Tomcat

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200914234320.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915010531.png)

- 检查异常攻击痕迹Filter/Servlet节点

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915011135.png)

**在Servlet节点中我发现到了自己设置的内存马test.ico，说明已经检测到了内存马**

## arthas

- 介绍

>arthas是Alibaba开源的Java诊断工具
https://github.com/alibaba/arthas

- 下载

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915153921.png)

- 文档地址

**非常Nice的工具，深入用法请查看使用文档，这里只检测探测一下**

```
https://arthas.aliyun.com/doc/quick-start.html
```

- 启动（选择对应tocmat进程pid）

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915160837.png)

- mbean(查看 Mbean 的信息，查看异常Filter/Servlet节点)

```
mbean | grep "Servlet"
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915161611.png)

- sc (查看JVM已加载的类信息)

```
sc xxx.* 模糊搜索类
sc -d
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915183918.png)

**查看payload加载的类信息**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915184214.png)

**查看x.AES_BASE64类加载的类信息**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915185544.png)

- jad(反编译指定已加载类的源码)

```
jad 类名
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915174856.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915174938.png)

**还有很多用法慢慢学习～**

## Copagent

**由于VisualVM在环境中可能还需要配置JVM Connection远程调试，我在亭一篇文章中发现了LandGrey师傅所写的内存马检测工具，经过在本地Tomcat测试，可以检测到我自己设置的内存马，而无需重启Tomcat服务（重启了内存马不就没了吗?）先贴上Git地址**


```
https://github.com/LandGrey/copagent
```

**我本地运行Tomcat服务，使用cop.jar工具，工具首先会识别你正在运行的应用列举出来由你自己选择ID，运行后会在.copagent目录生成结果**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915005424.png)

**在输出结果中，可以查看异常类，例如我的1.jsp和X.AES_BASE64，他会显示所有运行的类以及危险等级，比较高的可以进入目录查看代码进行分析**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915121700.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915005447.png)

**然后在java或class文件夹会保存木马以及运行的类**


![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915121748.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200915122054.png)

## 参考

```
https://mp.weixin.qq.com/s/DRbGeVOcJ8m9xo7Gin45kQ
https://qiita.com/shimizukawasaki/items/5dc9fe780ffbf3a7699c
```