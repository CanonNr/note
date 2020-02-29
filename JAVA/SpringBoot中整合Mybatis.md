## 简介

**MyBatis** 本是Apache的一个开源项目iBATIS。

iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的**持久层框架**。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAOs）

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Ordinary Java Object,普通的 Java对象)映射成数据库中的记录



## 导入相关依赖

```xml
<properties>
  <mybatis.version>2.1.1</mybatis.version>
</properties>

<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>${mybatis.version}</version>
</dependency>
```



## 配置

```yml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
  mapper-locations: classpath:mapping/*Mapper.xml
  #config-location: classpath:mapping/config.xml # 主配置文件
  type-aliases-package: com.lksun.library.entity # 作用:在mapper.xml中resultType属性就无须写完整的包路径了只需要写一个类名
```





## 实体类

```java
package com.lksun.library.entity;

import javax.persistence.*;
import java.math.BigInteger;
import java.util.Date;

public class Reader {
  
    private Integer id;

    private String name;

    private String password;

    private BigInteger mob;

    private Date create_time;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public BigInteger getMob() {
        return mob;
    }

    public void setMob(BigInteger mob) {
        this.mob = mob;
    }

    public Date getCreate_time() {
        return create_time;
    }

    public void setCreate_time(Date create_time) {
        this.create_time = create_time;
    }

    @Override
    public String toString() {
        return "Reader{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", password='" + password + '\'' +
                ", mob=" + mob +
                ", create_time=" + create_time +
                '}';
    }
}
```



## 启动类

```java
@SpringBootApplication
@ServletComponentScan
@MapperScan("com.lksun.library.Mapper") // 添加此注解 扫描所有mapper类
public class LibraryApplication {

    public static void main(String[] args) {
        SpringApplication.run(LibraryApplication.class, args);
    }

}
```



## Mapper类

```java
@Repository
//@Mapper  // 如果配置了MapperScan则不需要此注解了
public interface ReaderMapper {
		// xml式
    int createReader(Reader reader);

    List<Reader> get(PageRequest pageRequest);
		
  	// 注解式
    @Select("select * from readers where id = #{id}")
    Reader getById(@Param("id") Integer id);
}

```





## 业务层

### ReaderService

```java
package com.lksun.library.Service;

import com.lksun.library.entity.Page.PageRequest;
import com.lksun.library.entity.Reader;
import java.util.List;


public interface ReaderService {

    int createReader(Reader reader);

    List<Reader> get(PageRequest pageRequest);

    Reader getById(Integer id);
  
}

```



### ReaderServiceImpl

```java
@Service
public class ReaderServiceImpl implements ReaderService {

    @Autowired
    private ReaderMapper readerMapper;

    @Override
    public int createReader(Reader reader) {
        return readerMapper.createReader(reader);
    }

    @Override
    public List<Reader> get(PageRequest pageRequest) {
        System.out.println(pageRequest);
        return readerMapper.get(pageRequest);
    }

    @Override
    public Reader getById(Integer id) {
        return readerMapper.getById(id);
    }
}

```





## 映射文件

```xml
<!-- 路径和文件名注意与yml配置中对应-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 命名空间与Mapper文件对应 -->
<mapper namespace="com.lksun.library.Mapper.ReaderMapper">

    <select id="get" resultType="Reader">
        select * from readers
    </select>

</mapper>

```





## 控制器

```java
@Autowired
private  ReaderService readerService;

@RequestMapping(value = "",method = RequestMethod.GET)
public String list(PageRequest pageRequest, Map<String,Object> map){
  System.out.println(pageRequest);
  List<Reader> readerList = readerService.get(pageRequest);
  map.put("list",readerList);
  return "/reader/index";
}

@RequestMapping(value = "get/{id}",method = RequestMethod.GET)
public String getReader(@PathVariable("id") Integer id){
  Reader reader = readerService.getById(id);
  System.out.println(reader.toString());
  return "/reader/index";
}
```





## 其他

[MyBatis官方中文文档](https://mybatis.org/mybatis-3/zh/)