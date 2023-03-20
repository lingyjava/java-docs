# 6·Filter

- [6·Filter](#6filter)
  - [Filter是什么](#filter是什么)
  - [使用方式](#使用方式)
  - [字符编码过滤器](#字符编码过滤器)
  - [登录过滤器](#登录过滤器)

## Filter是什么
通过Filter可以拦截所有对Web服务器管理的任意Web资源的请求，从而实现特殊的控制功能。

使用Filter过滤器，实现url权限的控制，防止无权限用户进入某些页面，过滤敏感信息以及压缩响应格式等一些高级的功能。

## 使用方式
Filter在启动服务器时就已经创建。
每一个Filter必须执行chain.doFilter(request,response)方法,才能将控制权交到下一个过滤器或者目标资源。

## 字符编码过滤器

1. 在web.xml配置过滤器，过滤器应配置在所有servlet之前。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
<filter>
  <!--过滤器名-->
  <filter-name>encodingFilter</filter-name>
  <!--限定性类名-->
  <filter-class>com.java.filter.EncodingFilter</filter-class>
</filter>
  
<filter-mapping>
  <filter-name>encodingFilter</filter-name>
  <!--配置拦截那些路径-->
  <!--
		完全匹配：/index.jsp
		目录匹配：/admin/
		扩展名匹配：*.do
		全部匹配：/*
	-->
  <url-pattern>*.do</url-pattern>
</filter-mapping>
  
</web-app>
```

2. 使一个类成为过滤器。
```java
import javax.servlet.*;
/*设置字符编码,防止乱码*/

//可以通过注解完成在web.xml的配置
@WebFilter()
//继承Filter类,重写init(),doFilter(),destroy()方法
public class EncodingFilter implements Filter {
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("初始化过滤器");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //获取资源路径
        String url = request.getServletPath();
        
        //用来拦截请求，以及对拦截到的请求做业务逻辑处理
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        request.setCharacterEncoding("utf-8");
        response.setCharacterEncoding("utf-8");
        
        //当请求处理完成或符合要求后放行
        filterChain.doFilter(request,response);
    }

    @Override
    public void destroy() {
        System.out.println("过滤器销毁");
    }
}
```
## 登录过滤器

1. 在web.xml配置过滤器，过滤器应配置在所有servlet之前。
```xml
<filter>
  <filter-name>userIsLoginFilter</filter-name>
  <filter-class>com.java.filter.UserIsLoginFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>userIsLoginFilter</filter-name>
  <url-pattern>*.do</url-pattern>
  <url-pattern>*.jsp</url-pattern>
</filter-mapping>
```

2. 使一个类成为过滤器
```java
import javax.servlet.*;
/*判断用户是否登录的过滤器,防止无权限用户进行网页*/

@WebFilter()
public class EncodingFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("初始化过滤器");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //用来拦截请求，以及对拦截到的请求做业务逻辑处理
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
      	HttpSession session = request.getSession();
        //获取session域中的值
        User loginUser = (User) session.getAttribute("loginUser");
       	//获取资源路径,url的最后一节:/login.jsp
        String url = request.getServletPath();
        
        if(loginUser != null || "/login.jsp".equals(url) || "loginServlet.do".equals(url)){
            //说明登录了
            filterChain.doFilter(request,response);
        }else{
            //说明没有登录
            String basePath = request.getScheme();
            response.sendRedirect("login.jsp");
    }

    @Override
    public void destroy() {
        System.out.println("过滤器销毁");
    }
}
```
