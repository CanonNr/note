**在传统单机web应用中，一般使用tomcat/jetty等web容器时，用户的session都是由容器管理。浏览器使用cookie中记sessionId，容器根据sessionId判断用户是否存在会话session。这里的限制是，session存储在web容器中，被单台服务器容器管理**

### 导入jar包

```xml
<!--  cache 统一管理缓存类似JPA-->
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



### YML配置文件

```yml
spring:
  redis:
    host: localhost
    timeout: PT30M #30分钟
  session:
      store-type: redis
      timeout: PT30M
```



### 控制器中

```java
HttpSession session = request.getSession();
session.setAttribute("username","admin");

```

