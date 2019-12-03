## 1.第一步导入Jar包

```java
// pom.xml

// 通过此包已经将可能用到的core、web、context、beans、aop全部导入
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
	<version>4.3.5.RELEASE</version>
</dependency>

// 因为是WEB服务所以serverlet也少不了
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>javax.servlet-api</artifactId>
	<version>3.1.0</version>
</dependency>
```



## 2.配置web.xml文件

```xml

<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 设置配置文件 -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-servlet.xml</param-value>
    </init-param>
</servlet>

<servlet-mapping>
    <!-- 
		设置拦截:
 		把url-pattern元素的值改为 / ，表示拦截所有请求，由Spring MVC的后台控制器处理
	-->
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>

```



## 3.编辑 spring配置文件

```xml
<!--
 	配置1:
 -->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />

<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />

<!--  将 /hello 请求交给 MyController 控制器处理-->
<bean name="/hello" class="cn.lksun.controller.MyController"/>

<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>
```





## 4. 编写控制器

```java
package cn.lksun.controller;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

public class MyController implements Controller {
    
    public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest request, javax.servlet.http.HttpServletResponse response) throws Exception {
        System.out.println("hello Spring-MVC");
        request.setAttribute("name","canon");
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.setViewName("/hello.jsp");
        return modelAndView;
    }
}

```



## 5. 编写视图

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>Hello Spring MVC</h1>

<h2>我是:${name}</h2>
</body>
</html>

```





## 6. 得到结果

访问`127.0.0.1/hello`后可以看到:

# Hello Spring MVC

## 我是:canon

