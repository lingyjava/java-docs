# 2·Mybatis标签

- [2·Mybatis标签](#2mybatis标签)
  - [xml模板](#xml模板)
  - [trim](#trim)
  - [where](#where)
  - [转义标签](#转义标签)

## xml模板
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace:必须是对应接口的限定性类名-->
<mapper namespace="com.java.dao.EmpDao">

  <!--配置sql语句，给接口每个方法都配置一条sql语句-->

    <!--
        id必须是对应方法的名字
        parameterType:方法参数类型,传入参数,可以没有
        resultType:让mybatis自动把结果集封装成什么对象,返回类型,
				查询必须有返回值,增删改不需要指定返回值
        #{}：相当于占位符，可以获取参数的值
    -->
</mapper>
```

## trim
用来解决拼接update之类的动态SQL语句。
参数作用：

- `prefix`前缀
- `suffix`后缀
- `suffixOverrides`自动去掉多余的某个符号
```xml
<!--修改员工信息,姓名,上级领导,工资,入职时间,奖金,部门编号,岗位,头像-->
<update id="updateEmpByEmpNo" parameterType="emp">
  update emp
  <trim prefix="set" suffixOverrides=",">
    <if test="ename != null and ename != ''">
      ename = #{ename},
    </if>
    <if test="mgr != null and mgr != 0">
      mgr = #{mgr},
    </if>
    <if test="sal != null ">
      sal = #{sal},
    </if>
    <if test="hireDate != null and hireDate !=''">
      hiredate = #{hireDate},
    </if>
    <if test="comm != null ">
      comm = #{comm},
    </if>
    <if test="deptNo != null ">
      deptno = #{deptNo},
    </if>
    <if test="job != null and job !=''">
      job = #{job},
    </if>
    <if test="img != null and img != ''">
      img = #{img},
    </if>
  </trim>
  where empno = #{empNo}
</update>
```

## where
拼一个where关键字，去掉一个多余的and。
```xml
<!--多条件分页查询员工-->
<select id="queryByFy" parameterType="fy" resultType="emp">
  select * from (select e.*,rownum r from (select * from emp
  <where>
    <if test="query != null">
      <if test="query.qename != null and query.qename != ''">
        and ename like concat('%',concat(#{query.qename},'%'))
      </if>

      <if test="query.qjob != null and query.qjob != ''">
        and job like concat('%',concat(#{query.qjob},'%'))
      </if>

      <if test="query.qstartHireDate != null and query.qstartHireDate != ''">
        and HireDate > #{query.qstartHireDate}
      </if>

      <if test="query.qstopHireDate != null and query.qstopHireDate != ''">
        and HireDate <![CDATA[<]]> #{query.qstopHireDate}
      </if>
    </if>
  </where>
  ) e) where r >= #{pageRowStart} and r <![CDATA[<]]>= #{pageRowStop}
</select>

<!--根据当前的条件查询符合要求的记录总数-->
<select id="queryEmpCountByQuery" parameterType="query" resultType="int">
  select count (1) from emp
  <where>
    <if test="qename != null and qename != ''">
      and ename like concat('%',concat(#{qename},'%'))
    </if>

    <if test="qjob != null and qjob != ''">
      and job like concat('%',concat(#{qjob},'%'))
    </if>

    <if test="qstartHireDate != null and qstartHireDate != ''">
      and HireDate > #{qstartHireDate}
    </if>

    <if test="qstopHireDate != null and qstopHireDate != ''">
      and HireDate <![CDATA[<]]> #{qstopHireDate}
    </if>
  </where>
</select>
```

## 转义标签
`<![CDATA[  ]]>`