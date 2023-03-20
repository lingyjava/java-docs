# 8·FreeMarker

- [8·FreeMarker](#8freemarker)
  - [FreeMarker是什么](#freemarker是什么)
  - [依赖 \& 配置](#依赖--配置)
  - [标签](#标签)

## FreeMarker是什么
在做WEB开发的时候，我们不可避免的就是在前端页面之间进行跳转，中间进行数据的查询等等操作。我们在使用SpringBoot之前包括我在内其实大部分都是用的是JSP页面，可以说使用的已经很熟悉。但是我们在使用springBoot开发框架以后我们会发现一个致命的问题，就是SpringBoot对Jsp的支持可以说是惨不忍睹，官方推荐我们进行使用的是FreeMarker模板引擎进行。

使用FreeMarker模板引擎进行页面的渲染。使用HTML代替Jsp页面。

## 依赖 & 配置
1. 导入freemarker相关依赖。
2. 添加配置文件。

使用freemarker模板时可以在设置-->编辑器-->文件类型-->FreeMarker模板，添加通配符。*.html，模板数据语言html。

在HTML页面中仍然可以使用EL表达式，但不再能使用C标签。

## 标签
1. 获取pojo属性：定义pojo，存储到map。${pojo.attribute}
2. 循环：定义了List，存储到map。
```swift
<#list targetList as littleList>
  <#--循环遍历各个值-->
  ${littleList.attribute}
  <#--循环遍历下标-->
  ${littleList_index}
</#list>
```

3. 判断：定义了List，存储到map。
```swift
<#list targetList as littleList>
  <#if littleList_index%2 == 0>
    1
  <#else>
    2
  </#if>
  ${littleList.attribute}
</#list>
```

4. 日期处理：定义了日期，存储到map。

日期在Freemarker中显示时需要转换为String。
```swift
${date?date}  格式 xx-xx-xx
${date?time} 格式 xx:xx:xx
${date?datetime} 格式 xx-xx-xx xx:xx:xx
注意：可以自定格式
${date?string('yy/MM/dd HH:mm:ss')}
```

5. 空值处理：定义了变量var，存储到map。
```swift
${var!"defaultValue"}
    或者使用<#if>判断
<#if var??>
    var不是Null，var=${var}
    <#else>
    var是null
</#if>
```

6. 定义一个变量，可以在本页面中使用${assign}使用，
```swift
<#assign num=1/>
${num}
```
