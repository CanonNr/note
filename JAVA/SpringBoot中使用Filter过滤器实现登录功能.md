## 功能要求

初学阶段还没有连接数据库只做一个最简单的就好，找来了一个后台的模板，登录的账号密码直接单用户写死了，并判断Session没有登录自动跳转到Login界面。

Session直接改为了redis驱动

## 开始

### 导入相关的Jar包

```xml
<!--模板引擎-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<!--缓存-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!--redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!--spring-session-->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```



### yml配置

```yml
server:
  port: 8989
spring:
  redis:
    host: localhost
    timeout: PT30M #30分钟
  session:
      store-type: redis
      timeout: PT30M
  mvc:
    static-path-pattern: /static/**
  resources:
    static-locations:
      - classpath:/static/
```



### HTML代码

直接网上随便搜来一个带登录的后代模板，删除一些无用东西后直接丢`resoures/templates`中。

因为方便后面路由的管理所有的静态文件都加上了`/static/...`  详情看上面的yml配置

修改相关html代码中的JS、CSS路径



### User类

创建一个很普通的User类

```java
package com.lksun.library.Domain;

public class User {
    private String username;
    private String password;

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

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}

```



### 控制器

主要包括了：

- 页面控制器
- 登录的处理接口
- 一个验证账号密码的方法

```java
package com.lksun.library.Controller;

import com.lksun.library.Domain.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.Map;

@Controller
@RequestMapping("/login")
public class LoginController {

    @RequestMapping(value = "",method = RequestMethod.GET)
    public String index(@ModelAttribute User user){
        return "login";
    }

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

    private Boolean validator(User user){
        String username = "admin";
        String password = "admin";
        return user.getUsername().equals(username) && user.getPassword().equals(password);
    }
}

```





### Filter过滤器

可能是最关键了吧

```java
package com.lksun.library.Filter;

import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.*;

@Order(1)  // 表示过滤器的排序 数字越小，优先级越高  如果不设置则按照过滤器名称字母排序
@WebFilter(filterName = "authFilter", urlPatterns = {"/*"})
@Component
public class AuthFilter implements Filter {
	
    // 这是两个数组记录了需要过滤的路由，一个是完整路径一个是泛路径
    // 肯定还有更优雅的写法咱不是初学者嘛，以后更新
    private static final Set<String> ALLOWED_PATHS = Collections.unmodifiableSet(new HashSet<>(
            Arrays.asList("/login", "/register")));
    private static final Set<String> FUZZY_ALLOWED_PATHS = Collections.unmodifiableSet(new HashSet<>(Arrays.asList("/login/**","/static/**")));

    /**
     * 可以初始化Filter在web.xml里面配置的初始化参数
     * filter对象只会创建一次，init方法也只会执行一次。
     * @param filterConfig
     * @throws ServletException
     */
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    /**
     * 主要的业务代码编写方法
     * @param servletRequest
     * @param servletResponse
     * @param filterChain
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        String path = request.getRequestURI();
        if(!(ALLOWED_PATHS.contains(path) || this.allowed(path))){
            // 获取session
            HttpSession session = request.getSession();
            Object user = session.getAttribute("user");
            if (user == null){
                // 需要登录
                response.sendRedirect("/login");  // 重定向  
                //request.getRequestDispatcher("/login").forward(request, response);  // 转发
            }else{
                filterChain.doFilter(servletRequest, servletResponse);
            }
        }else{
            filterChain.doFilter(servletRequest, servletResponse);
        }
    }

    /**
     * 在销毁Filter时自动调用。
     */
    @Override
    public void destroy() {

    }

    private Boolean allowed(String path){
        Boolean result = null;
        String[] split = path.split("/");
        try {
            result = FUZZY_ALLOWED_PATHS.contains("/"+split[1]+"/**");
        }catch (Exception e){
            result = false;
        }
        return result;
    }
}

```



#### 答疑

- 为什么加上`allowed`这个方法？

  因为在`WebFilter`注解中我过滤了所有的路由，其中包括了**登录页面及登录的接口**和很多**静态文件**，会导致不能登录或登录界面中样式缺失。

  

- 为什么在`doFilter`加了那么多的`filterChain.doFilter(servletRequest, servletResponse);`

  核心的问题在于发现未登录时需要**重定向**或**转发**但是！

  无论是`request.getRequestDispatcher("/login").forward(servletRequest, servletResponse);`还是`response.sendRedirect("/login");`都有一个重定向的意思使用了`response`

  然后`filterChain.doFilte(servletRequest, servletResponse)`又一次使用了`response`会导致报错。

  ```java
  // 报错信息：
  
  java.lang.IllegalStateException: Cannot call sendRedirect() after the response has been committed
  
  Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception
  
  ```

  

  

  ​	