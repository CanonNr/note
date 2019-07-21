## JDBC

>  JAVA Database Connectivity java 数据库连接
>
> SUN公司提供的一种数据库访问规则、规范, 由于数据库种类较多，并且java语言使用比较广泛，sun公司就提供了一种规范，让其他的数据库提供商去实现底层的访问规则。 我们的java程序只要使用sun公司提供的jdbc驱动即可。

### 使用JDBC的基本步骤

1. 注册驱动

   

   ```java
   DriverManager.registerDriver(new com.mysql.jdbc.Driver());
   
   //打开Driver源码发现：
   public class Driver extends NonRegisteringDriver implements java.sql.Driver {
       public Driver() throws SQLException {
       }
   
       static {
           try {
               DriverManager.registerDriver(new Driver());
           } catch (SQLException var1) {
               throw new RuntimeException("Can't register driver!");
           }
       }
   }
   
   /**
   *	有一个静态的代码块，因为静态方法在类引用时可直接执行
   *	所以可跳过实例化的步骤 直接引用即可
   */
   
   Class.forName("java.sql.DriverManager");
   ```

   

2. 建立连接

   

   ```java
   //建立连接 参数一： 协议 + 访问的数据库 ， 参数二： 用户名 ， 参数三： 密码。
   conn = DriverManager.getConnection("jdbc:mysql://localhost/student", "root", "root");
   ```

   

3. 创建statement

   

   ```java
   // 跟数据库打交道，一定需要这个对象 
   st = conn.createStatement();
   ```



4. 执行sql ，得到ResultSet'

   

   ```java
   String sql = "select * from student";
   rs = st.executeQuery(sql);
   ```



5. 遍历结果集

   

   ```java
   while(rs.next()){
       int id = rs.getInt("id");
       String name = rs.getString("name");
       int age = rs.getInt("age");
       System.out.println("id="+id + "===name="+name+"==age="+age);
   
   }
   
   ```



6. 释放资源

   ```java
   if (rs != null) {
       try {
           rs.close();
       } catch (SQLException sqlEx) { } // ignore 
       rs = null;
   }
   
   ```

   



