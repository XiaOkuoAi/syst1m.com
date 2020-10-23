---
title: "Javassist"
date: 2020-10-23T18:18:09+08:00
draft: false
tags: [Javassist]
categories: [Java]
comment: true
---
Javassist
<!--more-->

# Javassist

**Javaassist 就是一个用来 处理 Java 字节码的类库。它可以在一个已经编译好的类中添加新的方法，或者是修改已有的方法**

## 创建Class文件

- 引入Javassist库

```xml
<dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>3.25.0-GA</version>
        </dependency>
```

- 使用javassist来创建一个ChaBug类

```java
public class Javassist {

    public static void main(String[] args) throws Exception{
        // 获取javassist维护的类池
        ClassPool pool = ClassPool.getDefault ();
        // 创建一个空类
        CtClass cc = pool.makeClass ("Chabug");
        // 新增一个字段 private String name
        CtField param = new CtField(pool.get("java.lang.String"), "name", cc);
        //设置访问级别为private
        param.setModifiers (Modifier.PRIVATE);
        // 设置name初始值
        cc.addField (param,CtField.Initializer.constant ("Y4er"));
        //生成getter setter方法
        cc.addMethod(CtNewMethod.setter("setName", param));
        cc.addMethod(CtNewMethod.getter("getName", param));
        // 添加无参构造函数
        CtConstructor cons = new CtConstructor (new CtClass[]{},cc);
        cons.setBody ("{name = \"syle\";}");
        cc.addConstructor (cons);
        // 5. 添加有参的构造函数
        cons = new CtConstructor(new CtClass[]{pool.get("java.lang.String")}, cc);
        // $0=this / $1,$2,$3... 代表方法参数
        cons.setBody("{$0.name = $1;}");
        cc.addConstructor(cons);
        // 创建一个public方法printName() 无参无返回值
        CtMethod printName = new CtMethod(CtClass.voidType, "test", new CtClass[]{}, cc);
        printName.setModifiers(Modifier.PUBLIC);
        printName.setBody("{System.out.println($0.name);}");
        cc.addMethod(printName);
        //这里会将这个创建的类对象编译为.class文件
        cc.writeFile ("/Users/syst1m/");
    }
}
```

执行结束之后生成了ChaBUg.class 文件

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201023161843.png)

## 方法
**ClassPool需要关注的方法：**

1、`getDefault :` 返回默认的ClassPool 是单例模式的，一般通过该方法创建我们的ClassPool；
2、`appendClassPath, insertClassPath :` 将一个ClassPath加到类搜索路径的末尾位置 或 插入到起始位置。通常通过该方法写入额外的类搜索路径，以解决多个类加载器环境中找不到类的尴尬；
3、`toClass : `将修改后的CtClass加载至当前线程的上下文类加载器中，CtClass的toClass方法是通过调用本方法实现。需要注意的是一旦调用该方法，则无法继续修改已经被加载的class；
4、`get , getCtClass :` 根据类路径名获取该类的CtClass对象，用于后续的编辑。

**CtClass需要关注的方法：**

1、`freeze :` 冻结一个类，使其不可修改；
2、`isFrozen :` 判断一个类是否已被冻结；
3、`prune : `删除类不必要的属性，以减少内存占用。调用该方法后，许多方法无法将无法正常使用，慎用；
4、`defrost : `解冻一个类，使其可以被修改。如果事先知道一个类会被defrost， 则禁止调用 prune 方法；
5、`detach : `将该class从ClassPool中删除；
6、`writeFile :` 根据CtClass生成 .class 文件；
7、`toClass :` 通过类加载器加载该CtClass。
上面我们创建一个新的方法使用了CtMethod类。CtMthod代表类中的某个方法，可以通过CtClass提供的API获取或者CtNewMethod新建，通过CtMethod对象可以实现对方法的修改。

**CtMethod中的一些重要方法：**

1、`insertBefore :` 在方法的起始位置插入代码；
2、`insterAfter : `在方法的所有 return 语句前插入代码以确保语句能够被执行，除非遇到exception；
3、`insertAt : `在指定的位置插入代码；
4、`setBody :` 将方法的内容设置为要写入的代码，当方法被 abstract修饰时，该修饰符被移除；
5、`make `: 创建一个新的方法。

## 调用生成的类对象

###  通过反射的方式调用

```java
Object o = cc.toClass ().newInstance ();
        Method setName = o.getClass ().getMethod ("setName",String.class);
        setName.invoke (o,"syst1m");
        Method printName1 = o.getClass ().getMethod ("printName",String.class);
        printName1.invoke (o);
```

### 通过读取 .class 文件的方式调用

```java
    public static void main(String[] args)  throws Exception{
        ClassPool pool = ClassPool.getDefault ();
        pool.appendClassPath ("/Users/syst1m/");
        CtClass ChaBUgClass = pool.get ("com.syst1m.ChaBug");
        Object o = ChaBUgClass.toClass().newInstance();

        //反射
    }
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201023171531.png)

### 通过接口的方式

- 新建一个接口Team，将ChaBUg类的方法全部抽象出来

```java
public interface Team {
    void setName(String name);
    String getName();
    void printName();
}
```

- 实现

```java
   ClassPool pool = ClassPool.getDefault ();
        pool.appendClassPath ("/Users/syst1m/");
        CtClass team = pool.get ("com.syst1m.Team");
        CtClass chabug = pool.get ("com.syst1m.ChaBug");
        chabug.defrost ();
        chabug.setInterfaces (new CtClass[]{team});
        team o = chabug.toClass ().newInstance ();
        o.setName("aaa");
        System.out.println (o.getName());
        o.printName();
```

## 修改现有的类

- PersonService

```java
public void getPerson(){
        System.out.println("get Person");
    }

    public void personFly(){
        System.out.println("oh my god,I can fly");
    }
```

- Javassist

```java
 public static void main(String[] args) throws Exception{
        ClassPool pool = ClassPool.getDefault ();
        CtClass cc = pool.get ("com.syst1m.PersonService");
        CtMethod personFly = cc.getDeclaredMethod ("personFly");
        personFly.insertBefore("System.out.println(\"起飞之前准备降落伞\");");
        personFly.insertAfter("System.out.println(\"成功落地。。。。\");");
        //新增一个方法
        CtMethod ctMethod = new CtMethod(CtClass.voidType, "joinFriend", new CtClass[]{}, cc);
        ctMethod.setModifiers(Modifier.PUBLIC);
        ctMethod.setBody("{System.out.println(\"i want to be your friend\");}");
        cc.addMethod(ctMethod);

        Object person = cc.toClass().newInstance();
        // 调用 personFly 方法
        Method personFlyMethod = person.getClass().getMethod("personFly");
        personFlyMethod.invoke(person);
        //调用 joinFriend 方法
        Method execute = person.getClass().getMethod("joinFriend");
        execute.invoke(person);
    }
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201023180842.png)