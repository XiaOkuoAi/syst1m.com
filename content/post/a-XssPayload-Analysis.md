---
title: "A XssPayload Analysis"
date: 2019-07-05T14:51:16+08:00
lastmod: 2019-07-05T14:51:16+08:00
draft: false
tags: [代码学习]
categories: []
comment: true
---
一个XSSpayload的分析
<!--more-->
# 一个XSSpayload的分析

>前提：不能使用字母和数字，使用jsfuck加密发现过滤"+"号
- 原始Payload
```-[][`${`${{}}`[!![]<<!![]<<!![]|!![]]}${`${{}}`[!!{}<<![]]}${`${[][~[]]}`[!!{}<<![]]}${`${!![][~[]]}`[(!![]<<!![])|!![]]}${`${![][~[]]}`[!{}<<![]]}${`${![][~[]]}`[!!{}<<![]]}${`${![][~[]]}`[!!{}<<!![]]}${`${{}}`[!![]<<!![]<<!![]|!![]]}${`${![][~[]]}`[!{}<<![]]}${`${{}}`[!!{}<<![]]}${`${![][~[]]}`[!!{}<<![]]}`][`${`${{}}`[!![]<<!![]<<!![]|!![]]}${`${{}}`[!!{}<<![]]}${`${[][~[]]}`[!!{}<<![]]}${`${!![][~[]]}`[(!![]<<!![])|!![]]}${`${![][~[]]}`[!{}<<![]]}${`${![][~[]]}`[!!{}<<![]]}${`${![][~[]]}`[!!{}<<!![]]}${`${{}}`[!![]<<!![]<<!![]|!![]]}${`${![][~[]]}`[!{}<<![]]}${`${{}}`[!!{}<<![]]}${`${![][~[]]}`[!!{}<<![]]}`](`${`${!![][~[]]}`[!!{}<<![]]}${`${!![][~[]]}`[!!{}<<!![]]}${`${![][~[]]}`[(!![]<<!![])|!![]]}${`${![][~[]]}`[!!{}<<![]]}${`${![][~[]]}`[!{}<<![]]}(${!!{}<<![]})`)()-```

- 首先复制全部到谷歌中尝试运行
![](https://ae01.alicdn.com/kf/HTB1dqNxXoz1gK0jSZLeq6z9kVXaA.jpg)
**其实是构造了一个函数**

- 把代码分成三部分
![](https://ae01.alicdn.com/kf/HTB1eQVOXX67gK0jSZPfq6yhhFXar.jpg)

- 依次查看内容
![](https://ae01.alicdn.com/kf/HTB1OkxyXbj1gK0jSZFuq6ArHpXac.jpg)
- 实际执行的语句为
![](https://ae01.alicdn.com/kf/HTB1UZxzXkH0gK0jSZPiq6yvapXaD.jpg)

- 再次分割（拿第一部分为例）
![](https://ae01.alicdn.com/kf/HTB144xzXlr0gK0jSZFnq6zRRXXal.jpg)

- 取第一部分为例
![](https://ae01.alicdn.com/kf/HTB1akJyXoD1gK0jSZFGq6zd3FXaa.jpg)

**那么为什么会是C**

- 列出基础字符
![](https://ae01.alicdn.com/kf/HTB1lMlzXhn1gK0jSZKPq6xvUXXam.jpg)
- 峰回路转
![](https://ae01.alicdn.com/kf/HTB1lMlzXhn1gK0jSZKPq6xvUXXam.jpg)
**对比字母为C的和基础字符，发现一开始是一样的，那么后面的**`[!![]<<!![]<<!![]|!![]]`**是什么**
![](https://ae01.alicdn.com/kf/HTB10pdzXkT2gK0jSZPcq6AKkpXai.jpg)
**是一个数组的格式，结合基础字符查看，相当于**`${{}[5]`
**`${{}`内容为`[object Object]`**
**相当于*`[object Object]`里面的第五个字符，就是`c`

**总结**：==利用基础字符然后类似数组的方式提取出想要的字母去构造出一个完整的payload==

**Thank for Wfox**
