## 使用properties配置文件

1. 在src目录下底下声明一个文件 xxx.properties ，里面的内容吐下：

   ```java
   driverClass=com.mysql.jdbc.Driver
   url=jdbc:mysql://localhost/student
   name=root
   password=root
   ```

   



2. 在工具类里面，使用静态代码块，读取属性

   ```java
   static{
       try {
           //1. 创建一个属性配置对象
           Properties properties = new Properties();
           InputStream is = new FileInputStream("jdbc.properties"); //对应文件位于工程根目录
   
           //InputStream is = 		Demo.class.getClassLoader().getResourceAsStream("jdbc.properties");
           //导入输入流。
           properties.load(is);
   
           //读取属性
           driverClass = properties.getProperty("driverClass");
           url = properties.getProperty("url");
           name = properties.getProperty("name");
           password = properties.getProperty("password");
   
       } catch (Exception e) {
           e.printStackTrace();
       }
   }
   ```

   