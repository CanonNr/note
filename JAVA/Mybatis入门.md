## Mybatis简介

Mybatis是一种基于ORM模式的,作用于Dao层的轻量级框架.和Hibernate类似,也支持各种SQL语句,也支持存储过程和高级映射等操作.

* Mybatis优点

> Mybatis比Hibernate更为轻量级;
> Mybatis几乎消除了所有的JDBC代码和参数的手工设置.
> Mybatis具有比较强大的动态语句功能.而且Mybatis在JavaBean和表之间的映射关系建立方面,也更加的便捷灵活.

## 简单的入门

1. 导入需要的jar包

   ```java
   <dependency>
   	<groupId>org.mybatis</groupId>
   	<artifactId>mybatis</artifactId>
   	<version>3.4.6</version>
   </dependency>
   
   <dependency>
   	<groupId>mysql</groupId>
   	<artifactId>mysql-connector-java</artifactId>
   	<version>8.0.15</version>
   </dependency>
   ```

   

2. 创建配置文件

   ```java
   /*
   	/resources/jdbc.properties 
   	和之前的一样一个简单的数据库配置文件
   */
   driver=com.mysql.cj.jdbc.Driver
   url = jdbc:mysql://localhost:3306/test
   username=root
   password=root
   
   ```

   ```xml 
   <!--
   	\resources\SqlMapConfig.xml  mybatis的配置文件
   -->
   
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
           
   <configuration>
   	<!-- 引入上面写的配置文件 -->
       <properties resource="jdbc.properties"></properties>
       <environments default="mysql">
           <environment id="mysql">
               <transactionManager type="JDBC"></transactionManager>
               <dataSource type="POOLED">
                   <property name="driver" value="${driver}" />
                   <property name="url" value="${url}" />
                   <property name="username" value="${username}" />
                   <property name="password" value="${password}" />
               </dataSource>
           </environment>
       </environments>
   
       <mappers>
       	<!-- 下面的是注解式的映射 -->
           <mapper class="com.dao.UserDao"/>
           <!-- 还可以这样(但是需要创建一个相对性的映射文件) -->
           <mapper resource="UserDao.xml"/>
       </mappers>
   </configuration>
   ```

   

3. 定义Dao与User类

   ```java
   // 只写接口就可以 无需实现
   public interface UserDao {
       List<User> getAll();
   }
   
   public class User {
    private String id;
       private String userName;
       private String email;
       
       /*
       	....
       	....
       	
       	以及相应的set get方法就不粘上了
    	
       	
       */
   }
   ```
   
   
   
4. 写入SQL语句（两种方式）

   ```java
   // 注解式
   public interface UserDao {
       @Select("select * from users")
       List<User> getAll();
       
       User find();
   }
   ```

   ```xml
   <!-- xml文件 与MyBatis配置文件中的映射文件对应 -->
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
   "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="MyMapper">
       <!--
       <select id="getAll" resultType="com.Dao.UserDao">
             select * from users
       </select>
   	-->
       <select id="find" resultType="com.Dao.UserDao">
             select * from users where id = #{id}
       </select>
   </mapper>
   
   ```

5. 执行

   ```java
   public static void main(String[] args) throws IOException {
   	//构建sqlSessionFactory
      {
   		// 读取配置文件
   		InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
   		// 构建sqlSessionFactory
   		SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
   		SqlSessionFactory factory = builder.build(in);
      }
   	// 打开sqlSession会话，并执行sql
      {
   		// 获取sqlSession
   		SqlSession sqlSession = factory.openSession();
   		// 操作CRUD
   		UserDao mapper = sqlSession.getMapper(UserDao.class);
   		List<User> users = mapper.getAll();
   	   /*
   			还可以这么写,第二个参数是定义sql时的占位参数
   			User user = sqlSession.selectOne("MyMapper.find", 1);
        		System.out.println(user);
   	   */
   	}
   	// 输出
   	System.out.println(users);
   }
   /*
   	得到结果：
   	[User{id=1, username='222222222', email='', password=''}, User{id=2, username='2312312', email 。。。。
   */
   ```

   