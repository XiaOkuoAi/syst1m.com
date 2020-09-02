---
title: "JavaSe"
date: 2020-09-02T17:41:38+08:00
draft: false
tags: []
categories: [Java]
comment: true
---
JavaSe泛型、代理、注解与反射
<!--more-->

# JavaSe泛型、代理、注解与反射

## 泛型

- 泛型，参数化类型，分为泛型类、接口、方法

### 泛型类

```
class 类名称 <泛型标识> {
    private 泛型标识 var;
    ...
}
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200829221407.png)

### 泛型接口

```
public class chabug {
    public static void main(String[] args) {
        new testa().next();
    }

}

interface test<T>{
    public T next();
}

class testa implements test<String>{
    private String[] b  = new String[]{"a","b","c","d"};
    @Override
    public String next(){
        Random rand = new Random();
        System.out.println(b[rand.nextInt(3)]);
        return null;
    }
}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200829222427.png)

### 泛型方法

```
public class chabug {
    public static void main(String[] args) {
        out("28");
        out("永远滴神");
    }

    public static <T> void out(T t) {
        System.out.println(t);
    }
}

```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830124017.png)
### 泛型通配符

```
public void showKeyValue1(Generic<?> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```


### 泛型上下边界
- 在使用泛型的时候，我们还可以为传入的泛型类型实参进行上下边界的限制，如：类型实参只准传入某种类型的父类或某种类型的子类。为泛型添加上边界，即传入的类型实参必须是指定类型的子类型,泛型的上下边界添加，必须与泛型的声明在一起.

- 常规例子（会提示类型无法转换）

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200829231117.png)

- 使用上下边界

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200829231228.png)

- 疑惑

```
GenericHolder<Fruit> fruitHolder = new GenericHolder<Fruit>()
```
为何意思

解决：

```
Fruit fruit = new Fruit("水果");
fruitHolder.setObj(fruit)
```

**先实例化再传入对象，参考如下：**

[Java泛型再探——泛型通配符及上下边界](https://www.imooc.com/article/26009)


### 泛型数组

- 不可以

```
List<String>[] ls = new ArrayList<String>[10];  
```
- 可以

```
List<?>[] ls = new ArrayList<?>[10]; 
List<String>[] ls = new ArrayList[10];
```

## 代理

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830162847.png)

### 静态代理

抽象角色：一般会使用接口或抽象类来解决
真实角色：被代理的角色
代理角色：代理真实角色后，我们一般会做一些附属操作
客户：访问代理对象的人

- 抽象角色

```
//接口——》租房
public interface Rent {

    public void rent();
}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830165257.png)

- 真实角色

```
//真实角色->房东
public class Host implements Rent{
    @Override
    public void rent(){
        System.out.println("房东要出租房子");
    }
}
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830165318.png)

- 代理角色

```
package ChaBugStudy;

public class Proxy implements Rent{

      private Host host;

      public Proxy(){
            
      }
      public Proxy(Host host){
          this.host = host;
      }

      public void rent(){
            seeHouse();
            host.rent();
            hetong();
            fare();
      }

      //看房

      public void seeHouse(){
            System.out.println("中介带你看房");
      }

      public void hetong(){
            System.out.println("签合同吧！");
      }
      //收中介费

      public void fare(){
            System.out.println("这房子不错，阔绰的你决定给点小费");
      }

}
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830170321.png)

- 客户端访问代理角色

```
public class Clinet {

    public static void main(String[] args) {
        Host host = new Host();
        Proxy proxy = new Proxy(host);

        proxy.rent();
    }
}
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830170342.png)

- 运行一下看看效果

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830170453.png)

- 缺点

**一个真实对象需要产生一个代理**

### 动态代理（先留着看完反射再看）

 - 动态代理（这里是基于JDK动态代理）

**一个动态代理类可以代理多个类，只要是实现了同一个接口**

- 抽象角色

```
public interface Rent {

   public void rent();
}
```

- 真实角色

```
public class Host implements Rent{

    public void rent(){
        System.out.println("房东要出租房子！");
    }
}
```

- 代理角色

```
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class Proxyhandler implements InvocationHandler {

    //被代理的接口

    Rent rent;
    public void setRent(Rent rent){
        this.rent = rent;
    }

    //生成得到代理类

    public Object getProxy(){
        return Proxy.newProxyInstance(this.getClass().getClassLoader(),rent.getClass().getInterfaces(),this);
    }

    // 处理代理实例，并返回结果

    public Object invoke(Object proxy, Method method,Object[] args) throws Throwable{
        Object result = method.invoke(rent,args);
        return result;
    }
}
```

- 客户端（客户）角色

```
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class Clent {

    public static void main(String[] args) {
        //真实角色
        Host host = new Host();
        //代理角色

        Proxyhandler pih = new Proxyhandler();
        //通过调用程序处理角色来处理要调用的接口角色

        pih.setRent(host);

        Rent proxy = (Rent) pih.getProxy();

        proxy.rent();
    }
}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200902164704.png)

## 注解（Java.Annotation）

- 格式

```
@注释名
```

- 内置注解

```
@Override
```

**定义在java.lang.Override中，表示重写**

```
Deprecated 
```

**定义在java.lang.Deprecated中，表示废弃，不鼓励使用，使用时会多一条横线，但依旧可以使用**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830182027.png)

```
@SuppressWarnings 
```

**定义在java.lang.SuppressWarnings中，用来抑制编译时的警告信息**

使用前

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830182927.png)

使用后

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830182950.png)

### 元注解

- 作用

**注解其他注解**

- @Target(用于描述注解的使用范围)

```
@MYAnnotation
public class ZhuJie {

    @MYAnnotation
    public static void main(String[] args) {

    }


}

//METHOD 方法、TYPE 类
@Target(value = {ElementType.METHOD,ElementType.TYPE})
@interface MYAnnotation{

}
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830185849.png)

- @Retention(表示需要在什么级别保存该注释信息，用来描述注解的生命周期（SOURCE < CLASS < RUNTime）)

```
@Retention(value = RetentionPolicy.RUNTIME)
```

- @Document(说明该注释将被包含在javadoc中)
- @Inherited(说明该子类可以继承父类中的该注解)

### 自定义注解

- @interface(自定义注解关键字)

- 格式

```
public @interface 注解名{定义内容}
```

- 自定义注解

```
package ChaBugStudy;


import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Syst1m(Team = "ChaBUg",Founder = "Y4er",brother = "syle")
public class ZhuJie {
    @Syst1m
    public static void main(String[] args) {
        System.out.println("ChaBug.Org");
    }


}


@Target(value = {ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface Syst1m{

    String Team() default "ChaBug";
    String Founder() default "Y4er";
    String brother() default "Syle";

}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200830192315.png)

## 反射（Reflection）

- 正常方式

**引入需要的“包类”名称-）通过new实例化-〉取得实例化对象**

- 反射方式

**实例化对象->getclass()方法->取得完整的“包类”名称**

- 优缺点

可以实现动态创建对象和编译，体现出很大的灵活性。

对性能有影响，慢于直接操作

- 主要API

```
java.lang.class  代表一个类
java.lang.reflect.Method  代表类的方法
java.lang.reflect.Field 代表类的成员变量
java.lang.reflect.Constructor 代表类的构造器
```

- 获取class类的几种方式

**常用方法**

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200901184138.png)


```
package ChaBugStudy;

public class Reflection {
    public static void main(String[] args) throws ClassNotFoundException {
        Person person = new Student();

        //方式一：通过对象获得
        Class c1 = person.getClass();
        System.out.println(c1.hashCode());

        //方式二：forname获得

        Class c2 = Class.forName("ChaBugStudy.Person");
        System.out.println(c2.hashCode());

        //方式三 通过类名.class

        Class c3 = Person.class;
        System.out.println(c3.hashCode());

        //方式四 内置类型的包装类

        Class c4 = Integer.TYPE;
        System.out.println(c4.hashCode());

        //获取父类

        Class c5 = c1.getSuperclass();
        System.out.println(c5.hashCode());

    }
}

class Person{
    String name;

    public Person(){

    }

    public Person(String name){

    }

    @Override
    public String toString(){
        return name;
    }
}

class Student extends Person{
    public Student(){
        this.name = "学生";
        }
}


class Teacher extends Person{
    public Teacher(){
        this.name = "老师";
        }
}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200901201630.png)

- 哪些类型可以有Class对象

**
class
interface接口
[]数组
enum枚举
注解@interface
基本数据类型
void
**

```
    Class c1 = Object c1 = Object.class; //类
    Class c2 = Comparable.class; //接口
    Class c3 = String[].class; //数组
    Class c4 = Override.class; //注解
    Class c5= ElementType.class; // 枚举
    Class c6 = Integer.class; // 基本数据
    Class c7 = void.class; //void
    Class c8 = Class.class; //CLass
```

- 获取运行时类的完整结构

**获取类的名字**

```
c1.getName() 获取包名加类名
c1.getSimpleName() 获得类名
```

**获得类的属性**

```
Field[] field = c1.getFields() 只能获取public属性
fields = c1.getDeclaredFields() 找到全部的属性
```

**获得指定属性的值**

```
Field name = c1.getDeclaredField("name")
```

**获得类的方法**

```
Method[] methods = c1.getMethods(); 获得本类以及父类的全部public方法

methods = c1.getDeclaredMethods(); 获得本类的所有方法
```

**获得指定方法**

```
Method getname = c1.getMethod("getname",null)
Method serName = c1.getMethod("setname",String.class)
```


**获得构造器**

```
Constructor[] constructors = c1.getConstructors();
for (Constructor constructor:constructors){
    System.out.println(construcior)}
    
constructors = c1.getDeclaredConstructors();
```

**获得指定构造器**

```
c1.getDeclaredConstructor(String.class,int.class)
```

- 动态创建对象执行方法

**获得对象**

```
Class c1 = Class.forname("XXX")
```

**构造对象**

```
User user = (User) c1.nerInstance(); 无参构造器
```
通过构造器创建对象
```
Constructor constructor = c1.getDeclaredConstructor(String.class)
User user2 = (USer)constructor.nerInstance("Syst1m")
```

**反射调用普通方法**

```
User user3 = (User)c1.nerInstance();
Method serName = c1.getDeclaredMethod("setName",Strind.class);
setName.invoke(User3,"syst1m");
```

**反射操作属性**

不能直接操作私有属性，需要关闭安全检测
```
User user4 = (User)c1.nerInstance();
Field name = c1.getDeclaredFields();

name.setAccessible(true) //关闭权限检测
name.set(User4,"syst1m");
```

- 参考

```
https://blog.csdn.net/lililuni/article/details/83449088
```
