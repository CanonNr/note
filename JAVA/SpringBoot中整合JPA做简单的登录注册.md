## 前言

之前已经做了一个不连接数据库的登录这次是在原有的基础上改为通过数据库获取用户的数据。

只是很简单的获取和添加一下数据。

## 导入相关依赖Jar包

```xml
<!-- JPA -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- MySQL -->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
</dependency>
```



## 配置文件

```yml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://127.0.0.1:3306/demo
    driver‐class‐name: com.mysql.jdbc.Driver
```



## 创建实体类

```java
package com.lksun.library.entity;

import javax.persistence.*;

@Entity  // 告诉JPA这是一个实体类
@Table(name = "users")  // 自定义数据库对应
public class User {
  	//这是一个主键列
    @Id 
  	// 主键的生成策略,JPA会自动选择一个最适合底层数据库的主键生成策略
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Integer id;
		
    // 其他的字段与数据库对应上即可
    @Column(name = "username")
    private String username;

    @Column(name = "password")
    private String password;

    @Column(name = "email")
    private String email;

    @Column(name = "address")
    private String address;

    @Column(name = "birthday")
    private String birthday;


    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getBirthday() {
        return birthday;
    }

    public void setBirthday(String birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", email='" + email + '\'' +
                ", address='" + address + '\'' +
                ", birthday='" + birthday + '\'' +
                '}';
    }
}

```





## 数据访问层

```java
package com.lksun.library.Dao;

import com.lksun.library.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Integer> {
    User findByUsername(String userName);
}
```



**JpaRepository**下包含了很多类似于`findByUsername`的方法，初步使用先写一个最简单的。



## 控制器

### 登录

```java
@Controller
@RequestMapping("/login")
public class LoginController {
		// 注入
    @Autowired
    private UserRepository userRepository;
		
  	// 这是登录的首页只返回了thymeleaf的模板
    @RequestMapping(value = "",method = RequestMethod.GET)
    public String index(@ModelAttribute User user){
        return "login";
    }
		
  	// 登录的操作方法
    @RequestMapping(value = "action",method = RequestMethod.GET)
    public String action(HttpServletRequest request, @ModelAttribute User user, Map<String,Object> map){
        Boolean validator = this.validator(user);
        if(validator){
            // 登录成功
            HttpSession session = request.getSession();
            session.setAttribute("user",user.getUsername());
            map.put("message","登录成功，等待跳转...");
            map.put("url","/");
            return "common/success";
        }else{
            // 失败
            map.put("message","账号或密码错误，请重试...");
            map.put("url","/login");
            return "common/error";
        }
    }
		
    // 判断用户名密码的方法
    private Boolean validator(User user){
        // 获取Form表单传回来的用户名与密码
        String username = user.getUsername();
        String password = user.getPassword();
        // 之前定义的方法,通过用户名查询
        User byUsername = userRepository.findByUsername(username);
        // 判断传来的用户名密码与数据库中是否一致
        // 仅为demo密码未加密 实际项目切勿模仿
        return password.equals(byUsername.getPassword());
    }
}
```



### 注册

```java
@Controller
@RequestMapping("/register")
public class RegisterController {
		// 注入
    @Autowired
    private UserRepository userRepository;
		
  	// 页面的显示,因为前端代码没有使用th:field,方法中的参数都省了
    @RequestMapping("")
    public String index(){
        return "register";
    }
		
    // 注册的处理方法，没有使用对象的方式接收From表单数据
    @RequestMapping(value = "action",method = RequestMethod.GET)
    public String action(@RequestParam("username") String username, @RequestParam("password") String password, @RequestParam("confirmPassword") String confirmPassword, Map<String,Object> map){
        // 判断两次密码是否相同
        if (!password.equals(confirmPassword)) {
            map.put("message","两次密码不同请重试...");
            map.put("url","/register");
            return "common/error";
        }
        // 判断用户名是否重复
        User findUser = userRepository.findByUsername(username);
        if (findUser != null){
            map.put("message","用户名重复...");
            map.put("url","/register");
            return "common/error";
        }
        User user = new User();
        user.setUsername(username);
        user.setPassword(password);
        // 添加数据库
        userRepository.save(user);
        map.put("message","注册成功...");
        map.put("url","/login");
        return "common/success";
    }
}
```

