# 4·EL表达式

- [4·EL表达式](#4el表达式)
	- [EL表达式是什么](#el表达式是什么)
	- [使用方式](#使用方式)

## EL表达式是什么
表达式语言（Expression Language），称EL表达式，是Java中的一种特殊的通用编程语言，借鉴于JavaScript和XPath。主要作用是在Java Web应用程序嵌入到网页（如JSP）中，用以访问页面的上下文以及不同作用域中的对象，取得对象属性的值，或执行简单的运算或判断操作。EL在得到某个数据时，会自动进行数据类型的转换。
可以在Jsp页面获取各种（作用域以及表单以及cookie）值。进一步减少在Jsp页面的Java代码。
使用Servlet接收url请求后，再通过EL表达式与JSTL标签库可以使Jsp页面没有Java代码。

## 使用方式
```html
<%
	request.setAttribute("name","req值");
	session.setAttribute("name","session值");
	application.setAttribute("name","application值");
	request.setAttribute("emp","emp对象");
%>

<!--获取四大作用域中的值-->
${name}	//EL表达式默认从四大作用域中取值
${requestScope.name}	//从request作用域中取name
${sessionScope.name}	//从session作用域中取name
${applicationScope.name}	//从application作用域中取name

${emp}	//获取emp集合
${emp[1]}	//获取emp集合下标为1的元素
${emp[1].ename}	//获取emp集合下标为1的元素的属性，直接写属性名称

<!--获取表单中的值-->
${param.qname}	//获取表单中的qname值，如果没有找到值，显示为空字符串

<!--获取cookie中的值-->
${cookie.nameC.value}	//获取cookie中的值
${cookie.pwdC.value}	//获取cookie中的值

<!--在EL中可以使用对象的静态方法-->
//因为cookie中不能存汉字，在存储时已转码，取值时需解码
${URLDecoder.decode(cookie.nameC.value,'utf-8')}	

<!--在EL中可以进行简单的计算-->
${param.page+10}	//输出int类型
${param==10}	//输出bool类型
```

