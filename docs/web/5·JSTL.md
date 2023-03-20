# 5·JSTL

## JSTL是什么
JSP标准标签库（JSP Standard Tag Library）是Java EE网络应用程序开发平台的组成部分。
使用标签的形式代替Jsp页面的Java业务逻辑，减少页面中的Java代码。

## 使用方式
1. 导入jstl.jar包
2. 在Jsp页面导入标签库
```html
<!--c标签库-->
<%@ taglib uri="http://java.sun.com/jsp/jstl/core"	prefix="c"%>
<!--日期格式化标签库-->
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
```

3. C标签
```html
<!--向作用域中放key-value数据-->
<c:set var="creq" value="reqsss" scope="request"></c:set>
<c:set var="score" value="90"></c:set>

<!--以字符串的形式输出value-->
<c:out value="${creq}"></c:out>

<!--删除指定作用域中的数据-->
<c:remove var="${creq}" scope="request"></c:remove>

<!--通过两个标签拼接一个url-->
<c:url var="whereURL" value="mainServlet.do" >
	<c:param name="qename" value="张三"></c:param>
  <c:param name="qjob" value="开发"></c:param>
</c:url>

<!--单分支选择语句-->
<c:if test="${score>=90}">
  优秀
</c:if>

<!--多分支选择语句-->
<c:choose>
  <c:when test="${score>=90}">优秀</c:when>
  <c:when test="${score>=70}">良好</c:when>
  <c:when test="${score>=60}">及格</c:when>
  <c:otherwise>不及格</c:otherwise>
</c:choose>

<!--普通for循环-->
<c:forEach begin="1" end="10" var="i">
	${i}
</c:forEach>

<!--forEach循环-->
<c:forEach items="${emps}" var="e">
	${e.ename}
  ${e.sal}
</c:forEach>
```

4. 日期格式化标签
```html
<fmt:formatDate value="${e.hireDate}" pattern="yyyy年MM月dd日"/>
```
