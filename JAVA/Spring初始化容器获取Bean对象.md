# Spring初始化容器获取Bean对象

## 首先了解下Spring 的三种配置方式

1. 注解式
2. XML
3. Java配置

## 注解

###  1.万年不变的导入Spring的Jar包

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

### 

## 注解式(配置类)

上面方法中如果使用@Component就必须在类上使用@ComponentScan，但是有些场景下每个类都加上@ComponentScan，所以就有了配置类。具体代码也只是在注解式上加了一个新的文件。

### 创建一个AppConfig类

```java
package hello;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration  // 证明这个类是配置类
@ComponentScan  // 扫描之前定义的需要spring声明的类
public class AppConfig {
}

```



### 删除ApplicationSpring中的@ComponentScan，并修改Spring容器实例化方法

```java
package hello;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ApplicationSpring {
    public static void main(String[] args) {
        // 唯一的区别在这：实例化方法里的参数不再是当前的类而改成了上面新建的配置类
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        MessagePrinter printer = context.getBean(MessagePrinter.class);
        printer.printMessage();
        /*  
           得到的结果还是一样的：
           Hello MessagePrinter
           Hello MessageService
           hello lksun ~
       */
    }
}

```



## XML

### 1.万年不变的导入Spring的Jar包

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



### 2.MessagePrinter和MessageService

和注解式没太大改变只是将相应的引用以及@Autowired、@ComponentScan、@Component删掉

其他完全相同不再贴代码

### 3.创建一个xml文件

在IDEA中找到 `resources`文件夹 -> 右键 -> New->XML Configuration File -> Spring Config

如果没有找到`Spring Config`证明Maven没有配置或者Jar包有问题

自己创建一个空的xml文件不要直接使用，spring有一个固定的格式（可以手动粘贴）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
        bean元素：表示当前对象通过Spring容器进行管理
        id元素：作为对象的唯一标识
        class元素：被管理的类的全名
    -->
    <bean id="service" class="hello.MessageService"></bean>

    <bean id="printer" class="hello.MessagePrinter">
        <!--
            name的service 和 MessagePrinter类中的 service属性是对应的
            ref的service  和 id等于service的Bean对象是对应的
            property作用：成功的创建了MessagePrinter和MessageService的一个依赖关系
        -->
        <property name="service" ref="service"></property>
    </bean>

</beans>

```

### 4. 执行文件

```java
package hello;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ApplicationSpring {
    public static void main(String[] args) {
        // 初始化一个Spring容器，这里和注解式用的方法不同参数也不同
        // 方法为ClassPathXmlApplicationContext，参数为刚才创建的XML文件名
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        // 与注解式相同的是此处同样会有两个方法的构造输出
        // 相同的getBean方法，在容器中获取指定的方法
        MessagePrinter printer = context.getBean(MessagePrinter.class);
        // 执行
        printer.printMessage();
    }
}

```



