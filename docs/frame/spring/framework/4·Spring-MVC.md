# 4·Spring-MVC

- [4·Spring-MVC](#4spring-mvc)
  - [官方文档](#官方文档)
  - [什么是Spring-MVC](#什么是spring-mvc)
  - [分层结构](#分层结构)
  - [工作流程](#工作流程)
  - [Jar包 \& 依赖](#jar包--依赖)
  - [前端控制器配置文件](#前端控制器配置文件)
  - [MVC主配置文件](#mvc主配置文件)
  - [接收作用域参数](#接收作用域参数)
  - [接收网页请求](#接收网页请求)
  - [跳转页面](#跳转页面)
  - [接收表单数据](#接收表单数据)
  - [处理AJAX请求](#处理ajax请求)
  - [上传图片](#上传图片)
  - [全局异常处理](#全局异常处理)
  - [拦截器](#拦截器)

## 官方文档
[spring-mvc](https://books.didispace.com/spring-mvc-4-tutorial/)

## 什么是Spring-MVC
SpringMVC框架对控制层进行了优化，使程序的结构更好易扩展，易维护。SpringMVC更像是一种设计思想。

## 分层结构
SpringMVC对Web分层设计结构，将控制层从Servlet、Jsp改为了Controller、View、Model.
![图11-Mvc分层架构](/docs/images/图11-MVC分层架构.jpeg)

## 工作流程
1. url请求发送到前端控制器
2. 控制器将url交给处理器映射器。
3. 处理器映射器找到对应的处理器，生成处理器执行链(包括处理器对象和处理器拦截器)返回给前端控制器。
4. 根据处理器获取处理器适配器，执行处理器适配器的一系列操作。
5. 执行处理器，通过处理器适配返回ModelAndView给前端控制器。
6. 前端控制器将ModelAndView交给视图解析器。
7. 视图解析器解析后返回View给前端控制器。
8. 前端控制器对View进行渲染。
9. 前端控制器响应到页面。

![图12-MVC工作流程](/docs/images/图12-MVC工作流程.png)

## Jar包 & 依赖
相关Jar包和依赖。
1. IOC.
2. AOP.
3. MVC.

## 前端控制器配置文件
配置前端控制器（web.xml）。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
  
  <!--前端控制器,启动IOC容器-->
  <servlet>
    <servlet-name>springDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--servlet初始化参数-->
    <init-param>
      <!--加载配置文件-->
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring*.xml</param-value>
    </init-param>
    <!--tomcat启动时就创建该servlet对象,默认接收到请求时创建-->
    <load-on-startup>1</load-on-startup>
  </servlet>
  
  <!--指定哪些请求经过springMVC处理-->
  <servlet-mapping>
    <servlet-name>springDispatcherServlet</servlet-name>
    <!--如果是/,代表拦截除jsp外的所有请求-->
    <url-pattern>*.do</url-pattern><!---->
  </servlet-mapping>
</web-app>
```

## MVC主配置文件
配置MVC主配置文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

  	<!--MVC提供的字符编码过滤器-->
    <filter>
        <filter-name>EncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <!--初始化参数-->
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>EncodingFilter</filter-name>
        <!--/*拦截所有请求-->
        <url-pattern>/*</url-pattern>
    </filter-mapping>
  
    <!--开启注解扫描-->
    <context:component-scan base-package="com.java"/>

    <!--开启springMVC的注解开发功能-->
    <mvc:annotation-driven/>

    <!--配置视图解析器-->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--拼接url-->
        <property name="suffix" value=".jsp"/>
        <!--前缀,可以通过/WEB-INF/访问到WEB-INF文件夹下的文件-->
        <property name="prefix" value="/"/>
    </bean>

    <!--排除静态资源-->
    <mvc:default-servlet-handler/>

</beans>
```

## 接收作用域参数
1. page作用域，代表本页面。
2. request作用域。
```java
import com.java.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

@Controller
//给该控制器里面方法的路径上设置父路径
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/login1.do")
    //在方法的参数列表使用HttpServletRequest参数获得request作用域
    public ModelAndView login1(HttpServletRequest request){
        //通过方法上注入的request对象,为作用域设置值
        request.setAttribute("键","值");

        //把需要设置到request中的数据设置到ModelAndView中
        ModelAndView mv = new ModelAndView();
        mv.addObject("键","值");
        //mv.setViewName()默认为请求转发
        mv.setViewName("index");
        return mv;
    }
    
    @RequestMapping("/login2.do")
    //通过在方法的参数中定义一个map集合,放到该集合中的数据默认放到request作用域
    public ModelAndView login2(Map<String,Object> args){
        args.put("键","值");
        return "main";//返回值默认是视图路径
    }

    @RequestMapping("/login3.do")
    //方法的对象类型的参数默认都会放到request作用域中,比如用于接收表单数据的对象
    //key默认是类名首字母小写
    public ModelAndView login3(User user){
        
        return "main";//返回值默认是视图路径
    }
}
```

3. Session作用域
```java
import com.java.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

@Controller
//给该控制器里面方法的路径上设置父路径
@RequestMapping("/user")
//将request作用域中的数据导入session作用域中
@SessionAttributes("user")
public class UserController {
    
    @RequestMapping("/login1.do")
    //在方法的参数列表使用HttpSession参数获得session作用域
    public ModelAndView login1(HttpSession session,HttpServletRequest request){
        //通过方法上注入的session对象,为作用域设置值
        session.setAttribute("键","值");
        
        //通过request获取
        request.setSession.setAttribute("键","值");

        ModelAndView mv = new ModelAndView();
        //mv.setViewName()默认为请求转发
        mv.setViewName("index");
        return mv;
    }
}
```

4. application作用域，通过参数注入的request或者session获取以及设置值。
```java
import com.java.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

@Controller
//给该控制器里面方法的路径上设置父路径
@RequestMapping("/user")
public class UserController {
    
    @RequestMapping("/login1.do")
    //在方法的参数列表使用HttpSession参数获得session作用域
    public ModelAndView login1(HttpSession session){
        //通过方法上注入的session对象,获取application作用域
        session.getServletContext().setAttribute("键","值");
        
        ModelAndView mv = new ModelAndView();
        //mv.setViewName()默认为请求转发
        mv.setViewName("index");
        return mv;
    }
    
}
```

## 接收网页请求
```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
//给该控制器里面方法的路径上设置父路径
@RequestMapping("/user")
public class UserController {

    //设置请求路径
    @RequestMapping("/login.do")
    //在jsp页面发出/user/login.do请求时,跳转到/login.jsp页面
    public ModelAndView login(){
        System.out.println("请求到达控制器");
        ModelAndView mv = new ModelAndView();
        //设置视图名称
        mv.setViewName("login");
        return mv;
    }
}

```

## 跳转页面
重定向是客户端重新发送了一个请求，所以不会经过视图解析器的包装，路径必须写根路径以下的全路径。

| 
 | 请求转发 | 重定向 |
| --- | --- | --- |
| 跳转到Jsp页面 | 默认请求转发到Jsp | view的名字必须加`redirect:`前缀。
 |
| 跳转到控制器
（不能直接写映射地址，因为会经过视图解析器的包装，变成.jsp） | view的名字必须加`forward:`前缀。 | view的名字必须加`redirect:`前缀。 |

前缀`forward:`相对路径，自动加上父路径
前缀`forward:/`绝对路径，从根路径开始
```java
@RequestMapping("/user")
public class UserController {
    @RequestMapping("/toLogin.do")
    public Stirng toLogin(){
        
        //跳转到login.jsp,相对路径请求转发
        return("login");//通过视图解析器的配置,拼接成/login.jsp
        //跳转到login.jsp,绝对路径请求转发
        return("/login");//'/'代表项目根路径
        
        //跳转到login.jsp,相对路径重定向
        return("redirect:login.jsp");//重定向不经过视图解析器,地址写全
        //跳转到login.jsp,绝对路径重定向
        return("redirect:/login.jsp");
        
        //跳转到login.do,相对路径请求转发
        return("forward:login.do");
        //跳转到login.do,绝对路径请求转发
        return("forward:/user/login.do");
        
        //跳转到login.do,相对路径重定向
		return("redirect:login.do");
        //跳转到login.do,绝对路径重定向
        return("redirect::/user/login.do");
        
        
    }

    @RequestMapping("/login.do")
    public String login(){

    }
}
```

## 接收表单数据
1. 通过和表单name属性相同的参数名或对象来获取普通表单数据。
```java
import com.java.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
//给该控制器里面方法的路径上设置父路径
@RequestMapping("/user")
public class UserController {

    //设置请求路径
    @RequestMapping("/login1.do")
    //直接通过和表单的name属性值一样的参数来接收，如果是多选框使用数组接收
    public ModelAndView login1(String name,String password,String isRemember){
        System.out.println(name+password+isRemember);

        ModelAndView mv = new ModelAndView();
        //设置视图名称
        mv.setViewName("index");
        return mv;
    }

    @RequestMapping("/login2.do")
    //通过对象类型的参数接收表单数据。
    public ModelAndView login2(User user){
        //springMVC自动把表单数据设置给和该表单name属性值一致的对象属性中
        //在对象中接收表单的多选框数据，必须使用集合
        System.out.println(user);
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }
}
```

2. 实体类中使用注解@DateTimeFormat()将时间格式化，接收日期类型数据。
```java
public class User implements HttpSessionBindingListener {
    private int id;
    private String name;
    private String password;
    //通过DateTimeFormat注解将时间格式化,并且做了非空处理
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date hireDate;
    //...
```

3. 接收对象中的对象类型参数的属性值，设置表单name值为对象类型值

如果没有给对象类型的属性传值，springMVC默认不会new对象类型属性。
```html
<form action="/springMVC_war_exploded/user/login2.do" method="post">
  用户名：<input type="text" name="name" id="name"><p></p>
  密码：<input type="password" name="pwd" id="password"><p></p>
  <!--使用对象中的对象属性值,name=对象.属性-->
  角色：<input type="text" name="role.name"><p></p>
  <input type="checkbox" name="isRemember" value="remember" checked="checked">记住用户名密码<p></p>
  <input type="submit" value="登录">
</form>
```

## 处理AJAX请求
1. 控制器方法返回一个Map集合或者需要响应给ajax的对象。
2. 对方法使用注解@ResponseBody表示该方法的返回json格式数据，结果不会被解析为跳转路径，直接写入 HTTP 响应正文中,一般在异步获取数据时使用。

## 上传图片
1. 导入jar包。
2. 配置文件上传解析器。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>

    <!--配置文件上传解析器-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!--默认编码-->
        <property name="defaultEncoding" value="utf-8"/>
        <!--最大字节-->
        <property name="maxUploadSize" value="1024000"/>
        <!--延迟解析,懒加载-->
        <property name="resolveLazily" value="true"/>
    </bean>
  
</beans>
```

3. 表单中图片上传框的name属性值不能和实体类中对应字段名一致，如果一致springMVC会自动进行设置，并且在对应的方法上。
4. 服务层处理图片。
```java
import com.java.pojo.Emp;
import com.java.service.FileUploadService;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.commons.CommonsMultipartFile;

import java.io.File;
import java.io.IOException;
import java.util.UUID;

@Service
public class FileUploadServiceImpl implements FileUploadService {

    @Override
    public Emp empFileUpload(Emp emp, CommonsMultipartFile avatar) {
        /*
         * 针对emp表的图片上传
         * */
        String oldImgName = avatar.getOriginalFilename();//获取图片原来的名字
        if (oldImgName != null && !"".equals(oldImgName)){
            String newImgName = UUID.randomUUID() +oldImgName.substring(oldImgName.lastIndexOf("."));//截取原名字后缀拼一个随机的新名字
            //上传框数据
            File imgFile = new File("d:\\Documents\\apache-tomcat-8.0.50\\webapps\\ssmLogin\\img\\"+newImgName);
            try {
                //把获取到的上传框数据复制到我们指定的新文件中
                avatar.transferTo(imgFile);
            } catch (IOException e) {
                e.printStackTrace();
            }
            emp.setImg("img/"+newImgName);
        }
        return emp;
    }
}
```

## 全局异常处理
springMVC不希望在各层的业务逻辑中出现try-catch，springMVC建议每层都想上抛出异常，然后在控制层做全局异常处理。

1. 让一个类作为全局异常处理器，根据不同的异常类型进行不同的处理。
```java
import org.springframework.stereotype.Controller;
import org.springframework.web.multipart.MaxUploadSizeExceededException;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Controller
public class GlobeException implements HandlerExceptionResolver {

    //全局异常处理器
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        ModelAndView mv = new ModelAndView();
        String path = httpServletRequest.getServletPath();
        System.out.println("当前异常路径:"+path);
        if (e instanceof ClassNotFoundException){
            System.out.println("异常:类没有找到");
            mv.addObject("error","异常:类没有找到");
            mv.setViewName("redirect:/error.jsp");//在MVC中即使用了重定向,error参数会拼接在url后传递
        }
        if (e instanceof MaxUploadSizeExceededException){
            System.out.println("异常：文件上传过大");
            mv.addObject("error","异常:文件上传过大");
            mv.setViewName("redirect:/error.jsp");
        }
        return mv;
    }
}
```

2. 出现异常时跳转到错误页面
```html
<body>
  <!--通过param获取error参数-->
    ${param.error}
</body>
```

## 拦截器
拦截器和过滤器的区别：
1. 过滤器拦截所有请求包括静态请求，拦截器只拦截控制层请求。
2. 过滤器是Servlet中的概念，拦截器是MVC中的概念。
3. 拦截所有请求时，过滤器配置文件使用'/*'，拦截器配置文件使用'/**'。

拦截器的实现：
1. 让一个类作为拦截器。
```java
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class UserInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        /*
         * 当一个请求到达,先顺序执行所有拦截器中的该方法
         * 只有返回true才会继续执行下一个拦截器的该方法
         * 返回false表示被拦截
         * */
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        /*
         * 当控制器方法执行完成,在return数据给用户之前执行
         * 拿到参数ModelAndView可以跳转页面
         * 可以处理数据,屏蔽违法信息等
         * */
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        /*
        * 当整个控制层方法结束后执行
        * 可以用于关闭资源等
        * */
    }
}
```

2. 配置拦截器，对应的拦截路径，执行顺序。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>
  
  <!--配置拦截器,拦截器顺序执行-->
    <mvc:interceptors>
        <!--第一个拦截器-->
        <mvc:interceptor>
            <!--
            拦截的路径
                /**:拦截所有请求
                /user:拦截父路径带user的请求
                /*.do:拦截以.do结尾的请求
            -->
            <mvc:mapping path="/**"/>
            <!--对应的拦截器-->
            <bean class="com.java.interceptor.UserInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>

</beans>
```
