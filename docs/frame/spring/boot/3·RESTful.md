# RESTful

- [RESTful](#restful)
  - [RESTful是什么](#restful是什么)
  - [幂等](#幂等)
  - [常用注解](#常用注解)

## RESTful是什么
REST（Representational State Transfer），表征性状态转移。

RESTFul是一种URL（统一资源定位符）的设计规范，在RESTFul中url只定位到资源，对于资源的操作使用提交方式区别。Spring Cloud推荐使用RESTFul格式的URL。使用这种风格的项目一般是前后端分离的项目，因为虽然URL提供了7种提交方式但是表单只支持POST和GET请求，照成普通页面发送DELETE以及PUT请求的时候需要复杂的处理。
RESTful架构，就是目前最流行的一种互联网软件架构。它结构清晰、符合标准、易于理解、扩展方便。因为"资源"表示一种实体，所以应该是名词，URI不应该有动词，动词应该放在HTTP协议中。

RESTFul的七种请求方式：
- Get：用来获取资源。
- Post：用来新建资源，也可用于更新资源。
- Delete：删除资源。
- Put：更新资源。
- Patch：用于创建、更新资源，代表部分更新。
- Options：用于url验证，验证接口服务是否正常。
- Teace：回显服务器收到的请求。
- Head：获得获取资源的部分信息。

## 幂等
幂等方法是指无论调用多少次都不会有不同结果的 HTTP 方法。它无论是调用一次，还是十次都无关紧要。结果仍应相同。
执行 10 次与执行 5 次产生不一样的结果，那么则不是一个幂等的方法。
幂等对于构建一个容易（fault-tolerent）的 API 非常重要。假设一个客户端希望通过 POST 来更新一个资源，由于 POST 是一个非幂等方法，调用它多次将导致错误的更新。那么当你向服务发送一个 POST 请求过程中超时将会发生什么，是不是资源已经被更新？超时发生在向服务器发送请求阶段还是服务返回客户端阶段？我们是否能够安全地重试一遍，或第一要查出资源究竟发生了什么变化？通过幂等方法，我们则无须回答这种问题，直到我们从服务器得到返回， 我们可以安全地重发请求。
幂等的请求方法：Get、Put、Delete、Options、Teace、Head
非幂等的请求方法：Post、Patch

## 常用注解
@RestController：相当于@Controller和@ResponseBody。
@RequestBody ：接收JSON格式字符串（对象）参数。
@PathVariable：接收路径参数。
@RequestParam：接收的参数是来自于请求头，也就是在url中。接收get请求参数的注解。
参数说明：
value：请求参数名。
required：该参数是否必需。默认为true。
defaultValue：请求参数的默认值。
@RequestBody：接收的参数是来自于请求体body中。接收post请求参数的注解。
@Null：限制只能为null。
@NotNull：限制必须不为null。
@AssertFalse：限制必须为false。
@AssertTrue：限制必须为true。
@DecimalMax(value)：限制必须为一个不大于指定值的数字。
@DecimalMin(value)：限制必须为一个不小于指定值的数字。
@Digits(integer,fraction)：限制必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction。
@Future：限制必须是一个将来的日期。
@Max(value)：限制必须为一个不大于指定值的数字。
@Min(value)：限制必须为一个不小于指定值的数字。
@Pattern(value)：限制必须符合指定的正则表达式。
@Size(max,min) ：限制字符长度必须在min到max之间。
@Past：验证注解的元素值（日期类型）比当前时间早。
@NotEmpty：验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0）。
@NotBlank：验证注解的元素值不为空（不为null、去除首位空格后长度为0），只应用于字符串且在比较时会去除字符串的空格。
@Email 验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式。
