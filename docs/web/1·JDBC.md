# 1·JDBC

- [1·JDBC](#1jdbc)
  - [JDBC是什么](#jdbc是什么)
  - [连接步骤](#连接步骤)
  - [加载Oracle驱动](#加载oracle驱动)
  - [加载MySQL驱动](#加载mysql驱动)
  - [获取数据库连接](#获取数据库连接)
  - [Statement | PreparedStatement](#statement--preparedstatement)
  - [获取结果](#获取结果)
  - [释放内存](#释放内存)
  - [使用规范](#使用规范)
  - [JDBCUtil工具类](#jdbcutil工具类)
    - [使用JDBCUtil](#使用jdbcutil)
    - [测试类](#测试类)

## JDBC是什么
JDBC（Java Database Connectivity），Java数据库连接，使用JDBC实现Java连接数据库。

## 连接步骤
```java
// 1. 获取JDBC驱动类
Class.forName("com.mysql.jdbc.Driver");
// 2. 获取数据库连接
Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);

// 3. 创建Statement或者PreparedStatement对象
Statement stmt = conn.createStatement();
// 4. 执行sql数据库查询
ResultSet rs = stmt.executeQuery("SELECT id, name, age FROM m_user where id =1");

// 5.解析结果集
System.out.println("name: "+ rs.getString("name") + " 年龄：" + rs.getInt("age"));
```

## 加载Oracle驱动
导入ojdbc6.jar包。
```java
//加载Oracle驱动
Class.forName("oracle.jdbc.driver.OracleDriver")
```

## 加载MySQL驱动
导入mysql驱动。
```java
//加载MySql驱动
Class.forName("com.mysql.jdbc.Driver")
```

## 获取数据库连接
```java
// MySql数据库连接
DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/mysql", user, pwd);

//Oracle数据库连接
DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:orcl", user, pwd);
```

## Statement | PreparedStatement
使用 `Statement` 或 `PreparedStatement` 对象发送SQL语句并带回执行结果。
- Statement：需要拼接字符串，容易导致sql注入，不推荐使用。
- PreparedStatement：存放待编译的语句。

```java
// 创建Statement对象
String sql = "select * from emp where empno = " + empno;
Statement st;
st = conn.createStatement();

// 创建PreparedStatement对象
String sql = "select * from emp where empno = ?";
PreparedStatement ps;
ps = conn.prepareStatement(sql);
// 给sql中的?参数赋值，1 代表第一个 ？
ps.getInt(1,empno);
```

## 获取结果
创建ResultSet集合存储。

```java
// 执行SQL获取查询结果，使用ResultSet存储
ResultSet rs = ps.executeQuery(sql);

// 如果是删除,添加,修改语句使用executeUpdate方法
rs = ps.executeUpdate(sql);
```

## 释放内存  
有序释放JDBC资源流。
```java
// 先打开的后关闭
if(rs!=null) {rs.close();}
if(st!=null) {st.close();}
if(conn!=null) {conn.close();}
```
## 使用规范
1.  一个Dao类只写一个表的增删改查。 
2.  对每个表提供对应pojo(实体)类，方便对表数据的处理。 
3.  每个Dao类需要提供对应的接口。 
4.  在写JDBC时不能使用Statement，拼接字符串时容易被sql注入。 
5.  如果查询多个返回List集合查询单个返回对应的实体类对象,如果是增删改返回int类型。 
6.  dao层的方法要求只能有一个参数。 
7.  必要时提供动态sql语句。 
8.  提供工具类，提供父类。 

## JDBCUtil工具类
```java
import java.sql.*;

public class JDBCUtil {
    private static JDBCUtil ju = new JDBCUtil();
    private JDBCUtil() {
        try {
            Class.forName("oracle.jdbc.driver.OracleDriver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    public static JDBCUtil init() {
        return ju;
    }

    /*提供公共的方法返回数据库连接*/
    public Connection getConnection() throws SQLException {
		Connection conn= DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:orcl", "lee", "tiger");
		return conn;
    }

    /*关闭资源*/
    public void close(Connection conn, PreparedStatement ps, ResultSet rs) {
        try {
            if(rs!=null) {
                rs.close();
            }
            if(ps!=null) {
                ps.close();
            }
            if(conn!=null) {
                conn.close();
            }
        } catch (Exception e2) {
        }
    }
}
```

### 使用JDBCUtil
EmpDaoImpl实现类示例。
```java
import com.example.jdbc.dao.EmpDao;
import com.example.jdbc.pojo.Emp;
import com.example.jdbc.util.JDBCUtil;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
public class EmpDaoImpl implements EmpDao {    
    private Connection conn;    
    private PreparedStatement ps;    
    private ResultSet rs;    
    @Override    
    public List<Emp> queryAll() {        
        String sql = "select * from emp";        
        try {            
            conn = JDBCUtil.init().getConnection();            
            ps = conn.prepareStatement(sql);            
            rs = ps.executeQuery();            
            /*把结果集中的数据封装成emp对象并放到List集合中*/            
            List<Emp> es = new ArrayList<Emp>();            
            while(rs.next()){                
                Emp emp = new Emp();                
                emp.setEmpNo(rs.getInt("empno"));                
                emp.setEname(rs.getString("ename"));                
                emp.setSal(rs.getDouble("sal"));
                emp.setComm(rs.getDouble("comm"));                
                emp.setDeptNo(rs.getInt("deptno"));                
                emp.setMgr(rs.getInt("mgr"));                
                emp.setJob(rs.getString("job"));                
                emp.setHireDate(rs.getDate("hiredate"));                
                es.add(emp);            
            }            
            return es;
        } catch (SQLException throwables) {            
            throwables.printStackTrace();        
        } finally {            
            JDBCUtil.init().close(conn,ps,rs);        
        }        
        return null;    
    }    
    @Override    
    public Emp queryEmpByEmpNo(int empNo) {        
        String sql = "select * from emp where empno = ? ";        
        try {            
            conn = JDBCUtil.init().getConnection();            
            ps = conn.prepareStatement(sql);            
            /*给占位符绑定值*/            
            ps.setInt(1,empNo);            
            rs = ps.executeQuery();            
            Emp emp = new Emp();            
            rs.next();            
            emp.setEmpNo(rs.getInt("empno"));            
            emp.setEname(rs.getString("ename"));            
            emp.setSal(rs.getDouble("sal"));            
            emp.setComm(rs.getDouble("comm"));            
            emp.setDeptNo(rs.getInt("deptno"));            
            emp.setMgr(rs.getInt("mgr"));            
            emp.setJob(rs.getString("job"));            
            emp.setHireDate(rs.getDate("hiredate"));            
            return emp;        
        } catch (SQLException throwables) {            
            throwables.printStackTrace();        
        }finally {            
            JDBCUtil.init().close(conn,ps,rs);        
        }        
        return null;    
    }    

    @Override    
    public int updateEmpByEmpNo(Emp emp) {        
        try {            
            conn = JDBCUtil.init().getConnection();            
            StringBuffer sql = new StringBuffer("update emp set ");            
            List<Object> args = new ArrayList<Object>();            
            if (emp.getSal() != null){                
                sql.append(" ,sal=?");                
                args.add(emp.getSal());            
            }            
            if (emp.getComm() != null){                
                sql.append(" ,comm=?");                
                args.add(emp.getComm());            
            }            
            if (emp.getDeptNo() != null){                
                sql.append(" ,deptno=?");                
                args.add(emp.getDeptNo());            
            }            
            if (emp.getJob() != null && !"".equals(emp.getJob())){                
                sql.append(" ,job=?");                
                args.add(emp.getJob());            
            }            
            sql.append(" where empno=?");            
            args.add(emp.getEmpNo());            
            sql.replace(sql.indexOf(","), sql.indexOf(",")+1, " ");            
            System.out.println(sql);            
            ps=conn.prepareStatement(sql.toString());            
            for(int i=0;i<args.size();i++) {                
                ps.setObject(i+1, args.get(i));            
            }            
            int num=ps.executeUpdate();            
            return num;        
        } catch (SQLException e) {            
            e.printStackTrace();        
        }finally {            
            JDBCUtil.init().close(conn,ps,rs);        
        }        
        return 0;    
    }    
    
    @Override    
    public int insertEmp(Emp emp) {        
        try {            
            String sql = "insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) values(?,?,?,?,?,?,?,?)";            
            conn = JDBCUtil.init().getConnection();            
            ps = conn.prepareStatement(sql);            
            ps.setInt(1,emp.getEmpNo());            
            ps.setString(2,emp.getEname());            
            ps.setString(3,emp.getJob());            
            ps.setInt(4,emp.getMgr());            
            ps.setDate(5,new java.sql.Date(emp.getHireDate().getTime()));//需要把util包下的Date转换成sql包下的Date            
            ps.setDouble(6,emp.getSal());            
            ps.setDouble(7,emp.getComm());            
            ps.setInt(8,emp.getDeptNo());            
            int num = ps.executeUpdate();            
            return num;        
        } catch (SQLException throwables) {            
            throwables.printStackTrace();        
        } finally {            
            JDBCUtil.init().close(conn,ps,rs);        
        }        
        return 0;    
    }    
    
    @Override    
    public int deleteEmpByEmpNo(int empNo) {        
        String sql = "delete from emp where empno = "+empNo;        
        try {            
            conn = JDBCUtil.init().getConnection();            
            ps = conn.prepareStatement(sql);            
            int num = ps.executeUpdate();            
            return num;        
        } catch (SQLException throwables) {            
            throwables.printStackTrace();        
        }  finally {            
            JDBCUtil.init().close(conn,ps,rs);        
        }        
        return 0;    
    }
}
```

### 测试类
```java
import com.example.jdbc.dao.EmpDao;
import com.example.jdbc.dao.daoImpl.EmpDaoImpl;
import com.example.jdbc.pojo.Emp;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
public class EmpDaoTest {    
    public static void main(String[] args) {        
        List<Emp> es = new ArrayList<Emp>();        
        Emp emp = new Emp();        
        emp.setEmpNo(250);        
        emp.setEname("loo");        
        emp.setSal(1000.0);        
        emp.setComm(250.0);        
        emp.setJob("king");        
        emp.setHireDate(new Date());        
        emp.setMgr(6666);        
        emp.setDeptNo(10);        
        EmpDao empDao = new EmpDaoImpl();        
        /*        
        es = empDao.queryAll();        
        for (int i = 0; i < es.size(); i++) {            
            System.out.println(es.get(i));        
        }        */        
        //System.out.println(empDao.queryEmpByEmpNo(6666));        
        //System.out.println(empDao.updateEmpByEmpNo(emp));        
        //System.out.println(empDao.insertEmp(emp));        
        //System.out.println(empDao.deleteEmpByEmpNo(250));    
    }
}
```
