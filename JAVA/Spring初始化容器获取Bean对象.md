# Spring初始化容器获取Bean对象

## 首先了解下Spring 的三种配置方式

1. 注解式
2. XML
3. Java配置

## 注解

###  1.Maven引入Spring的Jar包

在`pom.xml`文件中加入

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
</dependencies>
```



### 2. 在类中使用Component

举个栗子：

```java
package hello;

import org.springframework.stereotype.Component;

// 就是他↓
@Component
public class MessagePrinter {

    private MessageService service;
	
    // 此处加上了无参构造，主要用于理解Spring的自动实例化
    public MessagePrinter() {
        super();
        System.out.println("Hello MessagePrinter");
    }

    public void setService(MessageService service) {
        this.service = service;
    }
    void printMessage(){
        System.out.println(this.service.getMessage());
    }
}
```



```JAVA
package hello;

import org.springframework.stereotype.Component;

// 同理 ↓
@Component
public class MessageService {
    
    // 同理 ↓
    public MessageService() {
        super();
        System.out.println("Hello MessageService");
    }

    public String getMessage(){
        return "hello lksun ~";
    }
}

```



```java
package hello;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;

// 自动扫描之前定义的需要Spring声明的类
@ComponentScan
public class ApplicationSpring {
    public static void main(String[] args) {
		
        {
            // 这里是传统方式，需要手动new
            MessagePrinter printer = new MessagePrinter();
            MessageService service = new MessageService();

            printer.setService(service);
            printer.printMessage();
        }
        
        {
            // 通过Spring
            // 初始化一个Spring容器
            // 此步所有添加了@Component的类会自动执行构造方法,无需手动实例化
            ApplicationContext Context = new AnnotationConfigApplicationContext(ApplicationSpring.class);
            /*
           	 此时会得到两个类构造的输出
           	 Hello MessagePrinter
           	 Hello MessageService
         	*/
            // 在容器中获取定义的两个对象
            MessagePrinter printer = Context.getBean(MessagePrinter.class);
            MessageService service = Context.getBean(MessageService.class);
            // 执行
            printer.setService(service);
            printer.printMessage();
            // 得到输出：hello lksun ~
        }
    }
}

```



### Spring管理类与类的关系

在上面代码的基础上做了一点小改动

```java
// 在MessagePrinter类中的setService方法前加上@Autowired
// 作用在于：Spring已经知道了MessagePrinter和MessageService两个类的依赖和关联关系，所以自动的就可以执行set方法
@Autowired
public void setService(MessageService service) {
    this.service = service;
}

// 在ApplicationSpring类中删除set相关的方法
// 简化后为：
ApplicationContext Context = new AnnotationConfigApplicationContext(ApplicationSpring.class);
MessagePrinter printer = Context.getBean(MessagePrinter.class);
printer.printMessage();

/*
	此时得到的结果和之前的完全一样：
	Hello MessagePrinter
	Hello MessageService
	hello lksun ~
*/
```

