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

// 结果：[{id=1, name=小王, age=16}, {id=2, name=小刘, age=17}, {id=3, name=小刚, age=15}, {id=4, name=mm, age=50}, {id=5, name=周杰伦, age=50}, {id=6, name=周润发, age=32}, {id=7, name=彭于晏, age=28}]

// 修改操作
String sql = "UPDATE student set age=? where name='周杰伦'";
int res = template.update(sql,17);
System.out.println(res);
// 结果： 1
```



## RowMpper

```java
/*  
*	template中的方法可以创建List、Map...的结果,但是我们更需要的是对象
*	首先定义一个Student类(根据情况而定)
*/

// 首先还是写一个SQL语句
String sql = "SELECT * FROM student";
// 调用query方法,第一个参数是万年不变的SQL语句,第二个是一个RowMapper类
List<Student> list= template.query(sql,new RowMapper<Student>() {
    @Override
    public Student mapRow(ResultSet res, int i) throws SQLException {
        // 实例化已经准备好的Student类
        Student student = new Student();
        // 获取ResultSet返回的结果 并通过Student类的set方法定义变量
        student.setId(res.getInt("id"));
        student.setAge(res.getInt("age"));
        student.setName(res.getString("name"));
        // 输出对象
        return student;
    }
});

// 此刻我们获取到了一个List集合,遍历输出
for (Student student:list) {
    System.out.print(student.getId()+"  ");
    System.out.print(student.getName()+"  ");
    System.out.println(student.getAge()+"  ");
}

/**
* 结果：
*  	1  小王  16
*  	2  小刘  17
*  	3  小刚  15
*  	4  mm  50
*  	5  周杰伦  17
*  	6  周润发  32
*  	7  彭于晏  28
*/


// ↓↓↓ 当然还有简写 ↓↓↓
```



## BeanPropertyRowMapper

```java
String sql = "SELECT * FROM student";
List<Student> list= template.query(sql,new BeanPropertyRowMapper<Student>(Student.class));
for (Student student:list) {
    System.out.println(student.getName());
}
```

