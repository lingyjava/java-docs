# 3·Jsp

- [3·Jsp](#3jsp)
  - [Jsp是什么](#jsp是什么)
  - [Jsp标签](#jsp标签)
  - [请求响应对象](#请求响应对象)
  - [请求方式](#请求方式)
  - [跳转页面](#跳转页面)
  - [Cookie](#cookie)
  - [四大作用域](#四大作用域)
  - [九大内置对象](#九大内置对象)
  - [图片上传](#图片上传)
  - [处理中文乱码](#处理中文乱码)

## Jsp是什么
JSP（全称**J**akarta **S**erver **P**ages）是由Sun Microsystems公司主导创建的一种动态网页技术标准。JSP部署于网络服务器上，可以响应客户端发送的请求，并根据请求内容动态地生成HTML、XML或其他格式文档的Web网页，然后返回给请求者。JSP技术以Java语言作为脚本语言，为用户的HTTP请求提供服务，并能与服务器上的其它Java程序共同处理复杂的业务需求。
JSP文件在运行时会被其编译器转换成更原始的Servlet代码。JSP编译器可以把JSP文件编译成用Java代码写的Servlet，然后再由Java编译器来编译成能快速执行的二进制机器码，也可以直接编译成二进制码。
Jsp引擎：将整个Jsp页面解析成一个servlet类。使用out对象通过字符串拼接输出前端代码。

JSP将Java代码和特定变动内容嵌入到静态的页面中，实现以静态页面为模板，动态生成其中的部分内容。

## Jsp标签
1. <% %>：用于定义局部变量以及方法的调用。在标签内的代码相当于在一个方法中。
2. <%! %>：用于定义全局变量以及定义方法。
3. <%= %>：输出变量到页面。
4. <%@ %>：导包
 
## 请求响应对象
1. request(请求)：tomcat把当前数据封装成一个HttpServletRequest对象。
```java
/*获取表单数据*/
String name = request.getParameter("username");	//获取表单name属性为username的值
String[] hobbys = request.getParameterValues("hobby");
//通过request获得的数据都是String类型
```

1. response(响应)：tomcat把服务器想交给客户端的数据封装成一个HttpServletResponse对象。

## 请求方式
1. get：把表单数据全部以键值对的形式显示在地址栏，对地址的长度有限制。
2. post：不会在地址栏显示表单数据，并且对请求数据的长度没有要求。

## 跳转页面
在HTTP中有的特征。
无连接（一次连接与回复后断开连接）
无状态（当连接被回复后摧毁request对象）
在Jsp页面中，一个页面若想使用另一个页面的数据，需要用到请求转发，或重定向时在URL后拼接上数据。

1. 请求转发
如果两个页面需要进行大量数据传递，使用请求转发，两个页面共享request对象。
从a页面请求转发到b页面，地址栏仍是a页面的地址。
```java
request.getRequestDispatcher("b.jsp").forward(request,response);
```

2. 重定向
客户端在整个过程中发送了两次请求，两个页面不共享request对象。
从a页面重定向到b页面，地址栏为b页面的地址。
```java
response.sendRedirect("b.jsp");
response.sendRedirect("b.jsp?name=lee&pwd=tiger");	//重定向传数据
```

## Cookie
Web项目中用于保存用户数据的方式。将数据保存在用户机上。
经过用户同意以后，在服务器一端产生的数据通过服务器发送给客户端，并且以文本的形式保存在客户端，然后客户端每次和该服务器产生交互的时候都会把和该服务器相关的cookie发送给服务器。
Cookie默认无法保存中文,如果非要保存中文需要通过转码：
URLEncoder:把汉字转码成16进制的编码，
URLDecoder把16进制的编码转换成汉字。
使用步骤：
```html
<!--客户端接收Cookie-->
<form action="isLogin.jsp" method="post">
    用户名：<input type="text" name="name" id="name" value="${URLDecoder.decode(cookie.nameC.value,'utf-8')}"><p></p>
    密码：<input type="password" name="pwd" id="pwd" value="${cookie.pwdC.value}"><p></p>
    <input type="checkbox" name="isRemember" value="remember" checked="checked">记住用户名密码<p></p>
    <input type="submit" value="登录">
</form>
```
```java
/*创建cookie*/

//获取表单数据
String isRemember = request.getParameter("isRemember");

//代表记住用户名密码被选中，创建此cookie前先判断是否登录成功
if ("remember".equals(isRemember)){
	Cookie nameCookie=new Cookie("nameC", URLEncoder.encode(name, "utf-8"));
	Cookie pwdCookie = new Cookie("pwdC",pwd);
}

//设置Cookie有效时间
nameCookie.setMaxAge(10*24*60*60);
pwdCookie.setMaxAge(10*24*60*60);

//设置Cookie保存路径
nameCookie.setPath("/");
pwdCookie.setPath("/");

//发送给客户端
response.addCookie(nameCookie);
response.addCookie(pwdCookie);
```

## 四大作用域
1. page
作用域只在本页面有效，相当于Java中的全局变量。

2. request
放到request作用域中的数据只在本次请求有效，一般用来页面之间使用请求转发传递数据。
拼接在URL后的数据相当于表单数据，使用getParameter()获取这些数据。

3. session
会话，用于记录用户的登录状态。在一个session作用域中共享所有数据。
在服务器产生session对象，对象产生以后会生成一个sessionID保存在cookie缓存中，表示用户登录成功并开启一个会话，之后该客户端的每次请求都会带上sessionID，只有客户端发送的sessionID和服务器端保存的sessionID一致才表示在一个会话，否则会话结束。
session失效：用户关闭浏览器（缓存清除），客户端注销session（默认30分钟）
session在一定时间内将在服务器留下数据，所有除非必要不往session里面放数据（尤其是大量数据）

4. application
服务器一旦启动，就给每一个项目生成一个application对象，里面放置了关于web项目的一些配置信息，数据是所有客户端共享的。

## 九大内置对象
JSP有九个内置对象（又叫隐含对象），不需要预先声明就可以在脚本代码和表达式中随意使用。

1. request
获取客户端传递到服务器的信息。
getParameter():获得客户端传递给服务器一个参数的值。
getParameterNames():获得客户端传递给服务器的所有参数的名字。
getParameterValues():获取一个参数的所有值（如checkBox的情况）。setAttribute(),getAttribute(),removeAttribute():这三个方法主要用于struts框架中，必须在同一个请求中设置的属性才能获得。
getCookies():把个人信息存放到客户端。
setCharacterEncoding()/getCharacterEncoding():设置/获得字符编码。
getContextLength():获得整个网页的长度（长度不确定时返回-1）
getRequestURL():返回当前网页的地址（http://localhost:8080/项目名/具体页面.jsp）。getRequestURI():返回项目名/具体网页.jsp。
getMethod():获得网页提交的方法，默认为get，还可以设置为post。getRemoteAddr():获得远程地址。getRemoteHost():获得远程主机的名称。
getServerPort():端口号（一般默认是8080）。
getServletPath():/具体网页.jsp。
getContextPath():/项目名，获得的是上下文路径。getHeader(),getHeaders(),getHeaderNames():request.getHeader("Referer");获得来自的网页。

2. response
向客户端浏览器输出信息，对客户的请求进行响应。

3. out
向客户端输出信息，常用的有out.print();或out.println()

4. session
会话对象。每个用户的会话空间是隔离的，互不影响。

5. application
应用对象。Application、Session、request都可以
通过setAttribute来设置属性，
通过getAttribute来获取属性的值，但是它们的可见范围是不一样的。
Application对象所设置的属性不会过期，它在整个服务器运行过程中都是有效的，直到服务器重启。
Session对象所设置的属性只有在同一个session中可见。
request对象所设置的属性只有在同一次请求之间可见。
通过Application.getRealPath();可以获得其真实路径。

6. page
JSP网页在翻译时会转换成一个servlet（而此servlet是一个类）。
它是JSP网页本身，page对象是当前网页转换后的servlet类的实例。

7. config
一般用来配置指定的JSP参数。

8. exception
用exception对象获取错误信息。

9. pageContext
获取其它八大对象的句柄
pageContext.getOut(); //获得out对象的句柄
pageContext.getRequest(); //获得request对象的句柄
pageContext.getResponse(); //获得response对象的句柄
pageContext.getSession(); //获得session对象的句柄
pageContext.getServletContext();//获得application对象的句柄
pageContext.getServletConfig(); //获得config对象的句柄
pageContext.getException(); //获得exception对象的句柄
pageContext.getPage(); //获得page对象的句柄

## 图片上传
1. 将图片上传到本地服务器的硬盘中
2. 在将图片的相对路径存储到数据库中

## 处理中文乱码
1. 如果是post请求
request.setCharacterEncoding("utf-8");
response.setCharacterEncoding("utf-8");

2. 如果是get请求:也就是拼在URL后面的汉字
通过URLEncoder对象把汉字转成编码，通过URLDecoder对象把编码转成汉字。
通过修改tomcat的配置,使其get请求支持utf-8编码的汉字。
