---
title: "PHP Deserialization"
date: 2020-05-12T10:20:35+08:00
draft: flase
tags: [代码审计]
categories: []
comment: true
---
 PHP 反序列化漏洞学习
<!--more-->

# PHP 反序列化漏洞学习

## 反序列化基础

- 序列化

**序列化 (Serialization)是将对象的状态信息转换为可以存储或传输的形式的过程**

- JSON序列化

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511183934.png)

- PHP 序列化

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511193020.png)

- 各个字段的含义

```
O:4:"test":3:{s:10:"testflag";s:6:"Active";s:7:"*test";c;s:5:"test1";s:5:"test1";}
```

**O=> 代表这是一个对象
4=> 对象名占四个字符（test）
test => 对象名
3 => 对象有三个属性
s:10:"testflag" =>属性名
s:6:"Active" => 属性值
s:7:"*test"  =>属性名
s:4:"test" => 属性值
s:5:"test1" =>属性名
s:5:"test1" => 属性值**

- Puiblic权限

```
该是几个字符就是几个字符
```

- Private权限（私有权限）

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511200420.png)

**.test.flag**
**因为是私有的，所以会在flag前加自己的类名，成为八个字符，前后两个%00，成为10个字符**
**私有属性序列化的时候格式是%00类名%00属性名**

- Protected 权限属性

**%00*%00属性名**

- 反序列化

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511202151.png)

## 反序列化漏洞

- 魔术方法

**我们只可以控制属性，却不可以控制方法，所以需要魔术方法在符合条件时触发**

```
(1)construct()：当对象创建时会自动调用(但在unserialize()时是不会自动调用的)。
(2)wakeup() ：unserialize()时会自动调用
(3)destruct()：当对象被销毁时会自动调用。
(4)toString():当反序列化后的对象被输出在模板中的时候（转换成字符串的时候）自动调用
(5)get() :当从不可访问的属性读取数据
(6)call(): 在对象上下文中调用不可访问的方法时触发
```

-  __toString触发条件

```
(1)echo ($obj) / print($obj) 打印时会触发

(2)反序列化对象与字符串连接时

(3)反序列化对象参与格式化字符串时

(4)反序列化对象与字符串进行==比较时（PHP进行==比较的时候会转换参数类型）

(5)反序列化对象参与格式化SQL语句，绑定参数时

(6)反序列化对象在经过php字符串函数，如 strlen()、addslashes()时

(7)在in_array()方法中，第一个参数是反序列化对象，第二个参数的数组中有toString返回的字符串的时候toString会被调用

(8)反序列化的对象作为 class_exists() 的参数的时候
```

- 举例

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511205821.png)

## 利用 phar:// 拓展 PHP 反序列化的攻击面

- 原先条件

```
1.首先我们必须有 unserailize() 函数
2.unserailize() 函数的参数必须可控
```

### phar文件结构

- a stub

**前面内容随便，后面是固定的，表示一个标准的phar文件**
```
xxx<?php xxx; __HALT_COMPILER();?>
```
- a manifest describing the contents

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511212206.png)

- the file contents

```
这部分就是我们想要压缩在 phar 压缩包内部的文件
```

### 生成payload

- 创建一个合法的phar文件

```
<?php
    class TestObject {
    
    }

    @unlink("phar.phar");
    $phar = new Phar("phar.phar"); //后缀名必须为phar

    $phar->startBuffering();

    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub

    $o = new TestObject();

    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算

    $phar->stopBuffering();
?>
```

## Tips

- __wakeup

```
属性值大于原有属性数可以绕过
```

- 正则

```
if (substr($data, 0, 2) !== 'O:'
      && !preg_match('/O:\d:\/', $data))

```

**代码对data进行了判断，不可以为对象，0:X，X不可以为数字，绕过方法可以使用array数组绕过第一个，在X前面加+绕过第二个限制，搭达到到达反序列化方法的步骤。在__destruct销毁时会调用createCache方法写入文件，达成目的。**    
                          
## 参考

[一篇文章带你深入理解PHP反序列化漏洞](https://www.k0rz3n.com/2018/11/19/%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E5%B8%A6%E4%BD%A0%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3PHP%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/)