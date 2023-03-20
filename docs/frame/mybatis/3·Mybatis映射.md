# 3·Mybatis映射

- [3·Mybatis映射](#3mybatis映射)
  - [结果集映射](#结果集映射)
  - [@Param](#param)
  - [级联查询](#级联查询)
  - [实践](#实践)
    - [Map作为入参和回参](#map作为入参和回参)

## 结果集映射
- [mybatis文件映射之自定义返回结果集](https://www.programminghunter.com/article/2190359633/)
- [Map接收Mybatis的结果，字段1为Key，字段2为Val](https://blog.csdn.net/m0_46698142/article/details/105665154)
- [Mybatis返回类型和接收参数为Map类型](https://codeantenna.com/a/MFwaU5L90f)

## @Param
当参数较少时，可以使用@Param注解为Mapper层方法设置多参数，避免使用实体类，
```java
public interface mapper {
    Object getByIdAndCode(@Param("id") Long id, @Param("code") String code);
}
```

## 级联查询
1. 在一对多关系中，多的一方建立一的一方的实体类类型属性。在查询User的同时可以查询出其对应的Role实体类的属性。
```java
//User实体类
public class User{//映射
	private Integer id;
	private String u_name;
	private String u_pwd;
	private Integer age;
	/*
	 * 在多的一方建立一的一方的实体类类型属性,在查询User的同时可以查询出
     * 其对应的Role
	 */
	private Role role;
	private Integer flag;
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public Integer getAge() {
		return age;
	}
	public void setAge(Integer age) {
		this.age = age;
	}
	public Integer getFlag() {
		return flag;
	}
	public void setFlag(Integer flag) {
		this.flag = flag;
	}
	public Role getRole() {
		return role;
	}
	public void setRole(Role role) {
		this.role = role;
	}
	public String getU_name() {
		return u_name;
	}
	public void setU_name(String u_name) {
		this.u_name = u_name;
	}
	public String getU_pwd() {
		return u_pwd;
	}
	public void setU_pwd(String u_pwd) {
		this.u_pwd = u_pwd;
	}
	@Override
	public String toString() {
		return "User [id=" + id + ", u_name=" + u_name + ", u_pwd=" + u_pwd + ", age=" + age + ", role=" + role
				+ ", flag=" + flag + "]";
	}
}
//Role实体类
public class Role {
	private Integer r_id;
	private String r_name;
	/*
	 * 用于表示一个Role对应多个Power权限
	 */
	private List<Power> powers;
	public Integer getR_id() {
		return r_id;
	}
	public void setR_id(Integer r_id) {
		this.r_id = r_id;
	}
	public String getR_name() {
		return r_name;
	}
	public void setR_name(String r_name) {
		this.r_name = r_name;
	}
	public List<Power> getPowers() {
		return powers;
	}
	public void setPowers(List<Power> powers) {
		this.powers = powers;
	}
	@Override
	public String toString() {
		return "Role [r_id=" + r_id + ", r_name=" + r_name + ", powers=" + powers + "]";
	}
}
//Power实体类
public class Power {
	private Integer p_id;
	private String p_name;
	private String p_url;
	public Integer getP_id() {
		return p_id;
	}
	public void setP_id(Integer p_id) {
		this.p_id = p_id;
	}
	public String getP_name() {
		return p_name;
	}
	public void setP_name(String p_name) {
		this.p_name = p_name;
	}
	public String getP_url() {
		return p_url;
	}
	public void setP_url(String p_url) {
		this.p_url = p_url;
	}
	@Override
	public String toString() {
		return "Power [p_id=" + p_id + ", p_name=" + p_name + ", p_url=" + p_url + "]";
	}
}
```

2. 配置UserMapper映射文件，自定义映射关系。
```xml
<mapper namespace="com.zl.dao.UserDao">
	<!-- 自定义映射关系 -->
	<resultMap type="com.zl.pojo.User" id="userMap">
		<!-- 
		如果自定义映射map,那么最好把所有字段都进行映射,而不是只映射不一致的字段
		 -->
		 <id column="id" property="id"/><!-- 专门用来映射主键 -->
		 <result column="name" property="u_name"/>
		 <result column="pwd" property="u_pwd"/>
		 <result column="age" property="age"/>
		 <result column="flag" property="flag"/>
    
		 <!-- 级联查询User对象里面的Role对象 
		 	property:级联查询出来的对象设置给User里面的那个属性
		 	column:如果我要进行级联查询需要通过t_user里面的那个字段进行下一步查询:外键
		 	javaType:查询出来的对象是什么类型
		 	select:我需要执行那条sql语句
		 -->
		 <association property="role" column="role_id" javaType="com.zl.pojo.Role" select="com.zl.dao.RoleDao.queryUserByRoleId"/>
	</resultMap>

	<select id="login" parameterType="com.zl.pojo.User" resultMap="userMap">
		select * from t_user where name=#{u_name} and pwd=#{u_pwd}
	</select>
</mapper>
```

3. 配置RoleMapper映射文件。
```xml
<mapper namespace="com.zl.dao.RoleDao">
	<resultMap type="com.zl.pojo.Role" id="roleMap">
		<id column="id" property="r_id"/>
		<result column="name" property="r_name"/>
		<!-- 用于级联查询对象的List类型的属性
			property:查询出来的List集合设置给该对象的那个属性
			column:级联查询List集合的时候通过该字段值从Power表中查询
			ofType:List集合里面的数据类型
			select:要引用那条sql语句
		 -->
		<collection property="powers" column="id" ofType="com.zl.pojo.Power" select="com.zl.dao.PowerDao.queryPowerByRoleId"></collection>
	</resultMap>
	<!-- 必须给接口的每个方法配置一条sql语句,但是并不是每条sql语句都要对应方法 -->
	<select id="queryUserByRoleId" parameterType="int" resultMap="roleMap">
		select * from t_role where id=#{role_id}
	</select>
</mapper>
```

4. 配置PowerMapper映射文件。
```xml
<mapper namespace="com.zl.dao.PowerDao">
	<select id="queryPowerByRoleId" parameterType="int" resultType="com.zl.pojo.Power">
		select tp.id p_id,tp.name p_name,tp.url p_url from t_power tp,t_role_power trp 
			where tp.id=trp.role_id and trp.role_id=#{id}
	</select>
</mapper>
```

## 实践
### Map作为入参和回参
使用Map作为入参和回参可以节省实体类的创建，但降低了代码可读性。
