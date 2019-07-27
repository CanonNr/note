# JDBCTemplate

## 定义

Spring对数据库的操作在jdbc上面做了深层次的封装，使用spring的注入功能，可以把DataSource注册到JdbcTemplate之中。提供一个JDBCTemplate对象简化了队JDBC的开发。 

 ## 使用步骤

1. 导入Jar包

2. 创建JDBCTemplate对象（依赖于数据源DataSource）

   `JdbcTemplate template = new JdbcTemplate(dataSource);`

3. 调用JdbcTemplate 的方法完成数据库操作

   1. Update()：增删改操作
   2. queryForMap()
   3. queryForList()
   4. queryForObject()
   5. ...

## 操作

```java
// 加载配置文件
Properties p = new Properties();
p.load(jdbcTemplateTest.class.getClassLoader().getResourceAsStream("druid.properties"));
// 获取连接池对象
DataSource ds = DruidDataSourceFactory.createDataSource(p);
// 创建JdbcTemplate对象
JdbcTemplate template = new JdbcTemplate(ds);
// 定义SQL
String sql = "select  * FROM student";
// 执行SQL
List<Map<String, Object>> maps = template.queryForList(sql);
// 输出结果
System.out.println(maps);

//结果：[{id=1, name=小王, age=16}, {id=2, name=小刘, age=17}, {id=3, name=小刚, age=15}, {id=4, name=mm, age=50}, {id=5, name=周杰伦, age=50}, {id=6, name=周润发, age=32}, {id=7, name=彭于晏, age=28}]
```

