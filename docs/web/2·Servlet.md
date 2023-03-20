# 2·Servlet

- [2·Servlet](#2servlet)
  - [什么是Servlet](#什么是servlet)
  - [使用方式](#使用方式)
  - [生命周期](#生命周期)

## 什么是Servlet
Servlet（Server Applet），全称Java Servlet。是用Java编写的服务器端程序。其主要功能在于交互式地浏览和修改数据，生成动态Web内容。

Jsp在前端页面中插入Java代码的方式使网页难以维护。
Servlet是替代Jsp页面的Java代码，使用Servlet类在Java代码中接收前端页面的url请求。

## 使用方式

1. 定义一个类，继承HttpServlet类，使其变成Servlet类。
2. 重写doGet()和doPost()方法。
```java
@WebServlet("/helloServlet")//通过注解配置请求路径
public class HelloServlet extends HttpServlet{
    @Override
    protected void doGet(HttpRequest req,HttpResponse resq) throws ServletException{
        //如果是get请求执行
        doPost(req,resp);//将调用交给post
    }
    
    @Override
    protected void doPost(HttpRequest req,HttpResponse resq) throws ServletException{
        //如果是post请求执行
    }
    
}
```

3. 在web.xml中给该servlet绑定url路径。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
  <!--把项目中涉及到的servlet注册到此-->
  <servlet>
    <servlet-name>helloServlet</servlet-name>
      <servlet-class>com.java.action.HelloServlet</servlet-class><!--限定性类名-->
  </servlet>
  <!--给注册的servlet指定请求路径-->
  <servlet-mapping>
    <servlet-name>helloServlet</servlet-name>
      <url-pattern>/helloServlet.do</url-pattern><!---->
  </servlet-mapping>
</web-app>


```

4. 获得作用域。
```java
//request作用域   参数本身

//session作用域
HttpSession session = req.getSession;

//application作用域
//1.
ServletContext application = req.getServletContext();
//2.
req.getSession().getServletContext();
//3.
getServletContext();
```

5. 跳转页面。
```java
request.getRequestDispatcher("index.jsp").forward(request, response);//请求转发
response.sendRedirect("index.jsp");//重定向
```

## 生命周期
Servlet对象在被调用时创建。Servlet只创建一次（单例模式）。
当接收到对应的请求以后tomcat服务器帮我们创建该Servlet对象，创建之后不管处理多少次该请求，用的都是同一个Servlet对象（单例模式），所以不要在Servlet中定义全局变量，除非必要。
```java
public void init(){
	//当servlet对象创建以后第一个执行的方法
}

public void destroy(){
	//当servlet对象销毁前最后一个执行的方法
}
```
