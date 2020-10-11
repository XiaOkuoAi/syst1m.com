---
title: "JavaWeb"
date: 2020-10-11T15:43:45+08:00
draft: false
tags: [JavaWeb]
categories: [Java]
comment: true
---
JavaWeb
<!--more-->

# JavaWeb

## Servlet


**实现了Servlet接口的java程序叫做Servlet**

- 引入Servlet包

```
<dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
    </dependencies>
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200929152216.png)

- 新建moudle修改web.xml头信息

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
          http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
</web-app>
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200929160818.png)

### helloServlet

**编写代码**

```
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class helloservlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter writer = resp.getWriter ();
        writer.print ("hello,y4er");

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }


}
```

- 编写Servlet映射（web.xml中）

```
  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>helloservlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
```

- 运行测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200929162503.png)

### ServletContext

```
this.getInitParameter() 初始化参数
this.getServletConfig() Servlet配置
this.getServletContext() Servlet上下文
```

#### 共享数据

- helloservlet

```
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class helloservlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext ();
        
        String user = "syst1m";
        
        servletContext.setAttribute ("username",user);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }


}
```

- replyservlet

```
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class replyservlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext ();
        String username = (String) servletContext.getAttribute ("username");

        resp.setContentType ("text/html");
        resp.setCharacterEncoding ("utf-8");
        resp.getWriter ().print ("名字为"+username);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet (req, resp);
    }
}
```

- 注册Servlet

```
 <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>helloservlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
  <servlet>
    <servlet-name>reply</servlet-name>
    <servlet-class>replyservlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>reply</servlet-name>
    <url-pattern>/reply</url-pattern>
  </servlet-mapping>
```

- 运行测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200929171626.png)

#### 获得初始化参数

- 注册Servlet

```
  <context-param>
    <param-name>jdbc</param-name>
    <param-value>jdbc:mysql://localhost:3306/mybatis</param-value>
  </context-param>
```
  
```
 <servlet>
    <servlet-name>jdbc</servlet-name>
    <servlet-class>jdbcServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>jdbc</servlet-name>
    <url-pattern>/jdbc</url-pattern>
  </servlet-mapping>
```

- 代码

```
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class jdbcServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext ();
        String url = servletContext.getInitParameter ("jdbc");
        resp.getWriter ().print (url);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost (req, resp);
    }
}
```

- 运行测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200929181012.png)

#### 转发

- code 

```

public class forward extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext ();
        servletContext.getRequestDispatcher ("/jdbc").forward (req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost (req, resp);
    }
}
```

#### 读取资源文件

- db.properties

```
username=syst1m
password=123456
```

- code

```
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class jdbcServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        InputStream is = this.getServletContext ().getResourceAsStream ("WEB-INF/classes/db.properties");

        Properties prop = new Properties ();
        prop.load (is);
        String username = prop.getProperty ("username");
        String password = prop.getProperty ("password");

        resp.getWriter ().print (username+password);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost (req, resp);
    }
}
```

- 运行测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200929213530.png)

### HttpServletResponse

#### Response下载文件

- code 

```
public class FileServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取下载文件的路径
        String realPath = "/Users/syst1m/code/chabugservlet/com.web.servlet/src/main/resources/1.png";
        String FileName = realPath.substring (realPath.lastIndexOf ("\\")+1);
        resp.setHeader ("Content-Disposition","attachment;filename="+ URLEncoder.encode (FileName,"UTF-8"));
        FileInputStream in = new FileInputStream (realPath);
        int len = 0;
        byte[] buffer = new byte[1024];
        ServletOutputStream out = resp.getOutputStream ();
        while((len=in.read(buffer))>0){
            out.write (buffer,0,len);
        }

        in.close();
        out.close ();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

- 注册Servlet

```
</servlet-mapping>
  <servlet>
    <servlet-name>down</servlet-name>
    <servlet-class>FileServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>down</servlet-name>
    <url-pattern>/down</url-pattern>
  </servlet-mapping>
```

- 运行测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200930105034.png)


#### Response实现验证码

- code 

```
public class captchaServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //浏览器每三秒刷新一次
        resp.setHeader ("refresh","3");
        //内存中创建一张图片
        BufferedImage image = new BufferedImage (80,20,BufferedImage.TYPE_3BYTE_BGR);
        //得到图片
        Graphics2D g = (Graphics2D)image.getGraphics ();
        //设置图片背景色
        g.setColor (Color.white);
        g.fillRect (0,0,80,80);
        //给图片写数据
        g.setColor (Color.blue);
        g.setFont (new Font (null,Font.BOLD,20));
        g.drawString (makenum(),1,20);
        //浏览器：以图片类型打开
        resp.setContentType ("image/jpeg");
        //清除缓存，不让浏览器缓存
        resp.setDateHeader ("expires",-1);
        resp.setHeader ("Cache-Control","no-cache");
        resp.setHeader ("Pragme","no-cache");
        // 把图片写给浏览器
        ImageIO.write (image,"jpg",resp.getOutputStream ());

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
    //生成随机数

    private String makenum(){
        Random random = new Random ();
        String num = random.nextInt (9999)+"";
        StringBuffer sb = new StringBuffer ();

        for (int i = 0; i < 7-num.length (); i++) {
            sb.append ("0");
        }
        num = sb.toString ()+num;
        return num;
    }
}
```
- 注册Servlet

```
<servlet>
    <servlet-name>img</servlet-name>
    <servlet-class>captchaServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>img</servlet-name>
    <url-pattern>/img</url-pattern>
  </servlet-mapping>
```

- 运行测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200930141558.png)

#### Response实现重定向

```
resp.sendRedirect
```

### HttpServletRequest

- 注册Servlet

```
 <servlet>
    <servlet-name>LoginServlet</servlet-name>
    <servlet-class>LoginServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>LoginServlet</servlet-name>
    <url-pattern>/login</url-pattern>
  </servlet-mapping>
```

- index.jsp

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登陆</title>
</head>
<body>
<h1>登陆</h1>
    <div>
        <form action="${pageContext.request.contextPath}/login" method="post">

            用户名：<input type="text" name="username"><br>
            密码：<input type="password" name="password"><br>
            爱好：
            <input type="checkbox" name="hobbys" value="女孩">女孩
            <input type="checkbox" name="hobbys" value="代码">代码
            <input type="checkbox" name="hobbys" value="chabug">chabug
            <input type="checkbox" name="hobbys" value="渗透测试">渗透测试

            <br>
            <button type="submit">submit</button>
        </form>
    </div>
</body>
</html>  
```

- success.jsp

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登陆成功</title>
</head>
<body>
<h1>登陆成功</h1>
</body>
</html>
```

- LoginServlet

```
public class LoginServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       req.setCharacterEncoding ("UTF-8");
       resp.setCharacterEncoding ("UTF-8");
       String username = req.getParameter ("username");
       String password = req.getParameter ("password");
       String[] hobbys = req.getParameterValues ("hobbys");

       System.out.println (username);
       System.out.println (password);
       System.out.println (Arrays.toString (hobbys));
       req.getRequestDispatcher ("/success.jsp").forward (req,resp);


    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

- 运行测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200930215258.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200930220719.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200930220653.png)

### Cookie

- 注册Servlet

```
  <servlet>
    <servlet-name>cookie</servlet-name>
    <servlet-class>CookieServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>cookie</servlet-name>
    <url-pattern>/cookie</url-pattern>
  </servlet-mapping>
</web-app>
```
- code

```
public class CookieServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //解决乱码

        req.setCharacterEncoding ("UTF-8");
        resp.setCharacterEncoding ("UTF-8");
        PrintWriter out = resp.getWriter ();
        Cookie[] cookies = req.getCookies ();

        if(cookies!=null){
            out.write ("你上一次的时间是");
            for (int i = 0; i < cookies.length; i++) {
                Cookie cookie = cookies[i];
                if(cookie.getName ().equals ("lastlogintime")){
                    long logintime = Long.parseLong (cookie.getValue ());
                    Date date = new Date (logintime);
                    out.write (date.toLocaleString ());
                }
            }
        }else {
            out.write ("这是你第一次访问本站点");
        }
        Cookie cookie = new Cookie ("lastlogintime",System.currentTimeMillis ()+"");
        cookie.setMaxAge (***);
        resp.addCookie (cookie);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

- 运行测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201001155826.png)

### Session

- 设置，读取，删除

```
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		//获取Sesseion对象
		HttpSession session=request.getSession();
		session.setAttribute("username", "zhangsan");//存储数据
		session.getAttribute("username");//获取数据
		session.removeAttribute("username");//删除数据
	}
```

- Session的销毁

```
session.invalidate()
```

## JSP

- jsp原理

**内置对象**

```
final javax.servlet.jsp.PageContext pageContext; 页面上下文
javax.servlet.http.HttpSession session1 = null; session
final javax.servlet.ServletContext application; applicationContext
final javax.servlet.ServletConfig config; config
javax.servlet.jsp.JspWriter out = null; out 
final java.lang.Object page = this; page 当前
HttpServletRequest request  请求
HttpServletResponse response 响应
```

- maven依赖

**JSTL表达式依赖**

```
<dependency>
    <groupId>javax.servlet.jsp.jstl</groupId>
    <artifactId>jstl-api</artifactId>
    <version>1.2</version>
</dependency>
```

**standard标签库**
```
<dependency>
    <groupId>taglibs</groupId>
    <artifactId>standard</artifactId>
    <version>1.1.2</version>
</dependency>
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201002165412.png)

### JSP基础语法和指令

-  Tomcat热部署

```
https://blog.csdn.net/jc0803kevin/article/details/88579109
```

- Jsp表达式

```
<%--jsp表达式，用来将程序的输出，输出到客户端--%>
<%--<%= 变量或表达式%>--%>
<%= new java.util.Date()%>
```

- jsp脚本片段

```
<%

    int sum = 0;
    for (int i = 0; i < 100; i++) {
        sum+=1;
    }
    
    out.println ("<h1>sum="+sum+"</h1>");
%>
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201002175313.png)

- jso声明

```

<%!
    
    static {
        System.out.println ("Loading Servlet");
    }
    
    private int globalVar = 0;
    
    public void test(){
        System.out.println ("进入了方法test");
    }
%>
```

**jsp声明会被编译到JSP生成的java的类中，其他的会被生成到_jspServlet方法中**

### JSP指令

- 定制错误页面

**<%@page args...%>**

```
<%@page errorPage="err/err.html" %>
```

- 文件包含

```
<%@include file="index.jsp"%>
<jsp:include page="index.jsp">
```

### JSP内置对象及作用域

- 9大内置对象

```
PageContext  存东西
Request  存东西
Response
Session
Application  [ServletContext] 存东西
config  [ServletConfig]
out
page
exception
```

- code

```
 pageContext.setAttribute("name1","syst1m1");  //一个页面中有效
 request.setAttribute("name2","syst1m2"); //一个请求中有效
 session.setAttribute("name3"."syst1m3"); //一个会话中有效
 application.setAttribute("name4"."syst1m4"); //保存的数据在服务器中中有效
```

- 通过寻找的方式获取

```
pageContext.findAttribute();
```

- 重定向

```
pageContext.forward()
```

### Jsp、JSTL标签、EL表达式

#### EL表达式

- el表达式 ${}

**获取数据**

```
${标识符}
```

**执行运算**
**获取web开发的常用对象**

#### jsp标签

```
<jsp:forward page="test.jsp">
    <jsp:param name="paramtest" value="test"/>
</jsp:forward>
```

#### JSTL表达式

##### 核心标签

- 引入JSTL核心标签库

```
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```
- set标签：将值保存在指定的作用域中

```
<%-- var="变量名"  value="值" scope="保存在哪个作用域（page、request、session、application）"--%>
<c:set var="userName" value="yzq" scope="page"></c:set>
<span>${userName}</span><%--配合EL表达式取值--%>
```

- out标签：将结果输出

```
<%--取值--%>
<c:out value="${userName}"></c:out>
```

- if标签：判断

```
<c:if test="${age>20}">

    <span>奔三的人了</span>

</c:if>
```

- choose：选择，跟java中的switch语句类似

```
<c:set var="age" scope="page" value="40"></c:set>
<c:choose>

    <%--符合该条件时执行--%>
    <c:when test="${age>20&&age<30}">
        <span>奔三的人了</span>
    </c:when>

    <%--符合该条件时执行--%>
    <c:when test="${age<20}">
        <span>还是小鲜肉</span>

    </c:when>

    <%--之前条件都不满足时，执行这个--%>
    <c:otherwise>

        <span>可以考虑养生了</span>

    </c:otherwise>
</c:choose>
```

- forEach标签：用于迭代集合

```
<%--迭代标签 用于迭代集合--%>
<c:forEach items="${users}" var="user">

    <span>${user.name}</span>:<span>${user.age}</span>
    <br>

</c:forEach>
```

### JavaBean

**一般用来与数据库的字段做映射**


#### code

- javabean.java

```
package com.bean.pojo;

public class People {

    private int id;
    private String name;
    private int age;
    private String address;

    public People() {
    }

    public People(int id, String name, int age, String address) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.address = address;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "People{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", address='" + address + '\'' +
                '}';
    }
}
```

- javabean.jsp

```
<jsp:useBean id="people" class="com.javabean.pojo.People" scope="page"/>
<jsp:setProperty name="people" property="address" value="北京"></jsp:setProperty>
<jsp:getProperty name="people" property="address"/>
```


![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201009164549.png)


### 过滤器Filter

- 编写Servlet

```
public class helloservlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
         resp.getWriter ().write ("你好呀，世界");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

- 编写Filter

```
public class encodeing implements Filter {
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletRequest.setCharacterEncoding ("utf-8");
        servletResponse.setCharacterEncoding ("utf-8");
        servletResponse.setContentType ("text/html;charset=UTF-8");
        System.out.println ("过滤器执行前");
        filterChain.doFilter (servletRequest,servletResponse);
        System.out.println ("过滤器执行后");
    }

    public void destroy() {

    }
}
```

- 注册Filter

```
<filter>
    <filter-name>encode</filter-name>
    <filter-class>encodeing</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>encode</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

- 测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201010210744.png)


### 监听器

#### 统计在线人数Demo

- index.jsp

```
<html>
<body>
<h2>Hello World!</h2>


<h1>当前有<span><%=this.getServletConfig().getServletContext().getAttribute("OnlineCount")%></span></h1>
</body>
</html>
```

- OnlineCount

```
@Override
    public void sessionCreated(HttpSessionEvent httpSessionEvent) {
        ServletContext ctx = httpSessionEvent.getSession ().getServletContext ();
        Integer onlineCount = (Integer)ctx.getAttribute ("OnlineCount");

        if(onlineCount==null){
            onlineCount=new Integer (1);
        }else {
            int count= onlineCount.intValue ();
            onlineCount = new Integer (count+1);
        }
        ctx.setAttribute ("OnlineCount",onlineCount);
    }

    //销毁session监听
    @Override
    public void sessionDestroyed(HttpSessionEvent httpSessionEvent) {
        ServletContext ctx = httpSessionEvent.getSession ().getServletContext ();
        Integer onlineCount = (Integer)ctx.getAttribute ("OnlineCount");

        if(onlineCount==null){
            onlineCount=new Integer (0);
        }else {
            int count= onlineCount.intValue ();
            onlineCount = new Integer (count-1);
            httpSessionEvent.getSession ().invalidate ();
        }
        ctx.setAttribute ("OnlineCount",onlineCount);
    }

```

- 注册监听器

```
<listener>
    <listener-class>OnlineCount</listener-class>
</listener>
```

- 测试

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20201011114449.png)


## JDBC(Java fatabase connect)

### JDBC

```
package com.jdbc4;
  
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Timestamp;
import java.util.Date;
  
public class TestUpdate {
  
  /**
   * @param args
   */
  public static void main(String[] args) {
    Connection conn=null;
    try {
      Class.forName("com.mysql.jdbc.Driver");
      //建立连接  ctrl+shfit +o
      conn=DriverManager.getConnection("jdbc:mysql://localhost/test?useUnicode=true&characterEncoding=utf-8", "root", "19810109");
      String sql="update book set title=?,price=?,birth=?,publish_date=?,update_date=? where id=?";
      //预编译
      PreparedStatement pstmt=conn.prepareStatement(sql);
      pstmt.setString(1, "四集");
      pstmt.setFloat(2, 50.55f);
      pstmt.setDate(3, new java.sql.Date(new Date().getTime()));
      pstmt.setTimestamp(4, new Timestamp(new Date().getTime()));
      pstmt.setTimestamp(5, new Timestamp(new Date().getTime()));
      pstmt.setInt(6, 1);
        
      //执行
      pstmt.executeUpdate();
    } catch (Exception e) {
      e.printStackTrace();
    }
    finally
    {
      try {
        conn.close();
      } catch (SQLException e) {
        e.printStackTrace();
      }
    }
  }
  
}
  
```

### JDBC事务

```
connection.setAutoCommit(false) 开启事务
```