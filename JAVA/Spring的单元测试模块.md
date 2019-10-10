# Spring的单元测试模块

## 1.万年不变的引入Jar包

```xml
<dependencies>
    <!-- Spring的核心包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.1.10.RELEASE</version>
    </dependency>
	
    <!-- 单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <!-- Spring的测试模块 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.1.10.RELEASE</version>
    </dependency>

</dependencies>
```





## 2.开始

在`src`下有两个文件夹`main`和`test`。`main`是平时用的最多的，`test`就是测试的目录。

两个文件夹下都有个`java`文件夹。`main`和`test`下`java`的代码如果包名称相同物理路径不同但是最终编译后的代码是在一起的。

所以在`test.java`下创建一个`hello`的包 并在下面创建一个`AppTest`方法

```JAVA
package hello;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

// 自动初始化Spring容器，所以无需手动初始化
// 代替了ConfigApplicationContext方法
@RunWith(SpringJUnit4ClassRunner.class) 
// 加载配置类
@ContextConfiguration(classes = AppConfig.class) 
public class AppTest {
    
    // 自动注入
    @Autowired
    private MessagePrinter printer;
	
    // 添加Test的注解使此方法可以执行
    @Test
    public void action() {
        printer.printMessage();
    }
}

```