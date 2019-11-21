##  简介

Aspect-Oriented 编程(AOP)通过提供另一种思考程序结构的方式来补充 Object-Oriented 编程(OOP)。 OOP 中的 key 模块化单元是 class，而在 AOP 中，模块化单元是 aspect。方面实现了诸如 transaction management 之类的关注点的模块化，这些问题涉及多种类型和 objects。 (这种担忧通常被称为 AOP 中的横切关注 literature.)

Spring 2.0 引入了一种使用schema-based 接近或@AspectJ 注释风格编写自定义方面的更简单，更强大的方法。这两种样式都提供完全类型的建议和 AspectJ 切入点语言的使用，同时仍然使用 Spring AOP 进行编织。

### AOP 概念

让我们首先定义一些中心 AOP 概念和术语。这些术语不是 Spring-specific ......不幸的是，AOP 术语不是特别直观;然而，如果 Spring 使用自己的术语，那将更加令人困惑。

- 【**Aspect** 】面：跨越多个 classes 的关注点的模块化。 Transaction management 是企业 Java applications 中横切关注点的一个很好的例子。在 Spring AOP 中，方面是使用常规 classes schema-based 接近或使用`@Aspect` annotation@AspectJ 风格注释的常规 classes 实现的。

- 【**Join point**】 连接点：程序执行期间的一个点，例如方法的执行或 exception 的处理。在 Spring AOP 中，连接点始终表示方法执行。

- 【**Advice**】建议：aspect 在特定连接点采取的操作。不同类型的建议包括“周围”，“之前”和“之后”建议。 (建议类型被讨论 below.)许多 AOP 框架，包括 Spring，model 作为拦截器的建议，在连接点周围维护一系列拦截器。

- 【**Pointcut**】切入点：匹配连接点的谓词。建议与切入点表达式相关联，并在切入点匹配的任何连接点处运行(对于 example，执行具有特定 name 的方法)。由切入点表达式匹配的连接点的概念是 AOP 的核心，而 Spring 默认使用 AspectJ 切入点表达式语言。

- 【**Introduction**】简介：代表类型声明其他方法或字段。 Spring AOP 允许您向任何建议的 object 引入新接口(和相应的 implementation)。对于 example，您可以使用简介使 bean 实现`IsModified`接口，以简化缓存。 (引言在 AspectJ 中被称为 inter-type 声明 community.)

- 【**Target object**】目标 object：object 由一个或多个方面建议。也称为建议 object。由于 Spring AOP 是使用运行时代理实现的，因此 object 将始终是代理的 object。

- 【**AOP proxy**】AOP 代理：由 order 中的 AOP framework 创建的 object，用于实现 aspect contracts(建议方法执行等)。在 Spring Framework 中，AOP 代理将是 JDK 动态代理或 CGLIB 代理。

- 【**Weaving**】编织：将方面与其他 application 类型或 objects 链接以创建一个建议的 object。这可以在 compile time(使用 AspectJ 编译器，example)，load time 或运行时完成。与其他纯 Java AOP 框架一样，Spring AOP 在运行时执行编织。

  

### 通知方法：

   * 				前置通知(@Before)：logStart:在目标方法运行之前运行
   * 				后置通知(@After)：logEnd：在目标方法运行结束之后运行
   * 				返回通知(@AfterReturning)：logReturn：在目标方法正常返回之后运行
   * 				异常通知(@AfterThrowing)：logException：在目标方法出现异常之后运行
   * 				环绕通知：动态代理，手动推进目标方法运行（joinPoint.procced()）




### 简单来说四步走

1. 启用 @AspectJ 支持（@EnableAspectJAutoProxy ）
2. 将业务逻辑代码和切面类加入到Spring容器中，并声明切面类（@Aspect）
3. 在切面类中声明切入点（ @Pointcut ）
4. 声明通知（@Before、@After）

## 举一个栗子

我们现在有一个业务逻辑类**Say**(逻辑代码不需要关心)，我们的要求是在执行业务的同时顺便输出log日志（方法之前、方法运行结束、方法出现异常等）

- 首先导入jar包

  ```xml
  <dependencies>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>5.1.10.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.12</version>
          <scope>compile</scope>
      </dependency>
  
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-aspects</artifactId>
          <version>5.1.5.RELEASE</version>
      </dependency>
  </dependencies>
  ```

  

- 定义一个**Config**类

  ```JAVA
  package lksun.hello;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.EnableAspectJAutoProxy;
  
  // 启用 @AspectJ 支持
  @EnableAspectJAutoProxy
  // 配置spring并启动spring容器
  @Configuration
  // 添加自动扫描注解
  @ComponentScan
  public class Config {
     
  }
  
  ```

- 定义一个业务逻辑**Action**类

  ```java
  package lksun.hello;
  
  import org.springframework.stereotype.Component;
  
  @Component
  public class Action {
      // 随便写就好
      public String say(String message) {
          System.out.println(":)");
          /*
          	假如在此处添加错误代码 AfterThrowing 通知会执行
          	System.out.println(10/0);
          */
          return "Hello"+message+"!!";
      }
  }
  
  ```



- 定义一个**Log**的类并声明**Aspect**

  ```java
  package lksun.hello;
  
  import org.aspectj.lang.JoinPoint;
  import org.aspectj.lang.annotation.After;
  import org.aspectj.lang.annotation.AfterReturning;
  import org.aspectj.lang.annotation.AfterThrowing;
  import org.aspectj.lang.annotation.Aspect;
  import org.aspectj.lang.annotation.Before;
  import org.aspectj.lang.annotation.Pointcut;
  import org.springframework.stereotype.Component;
  import java.util.Arrays;
  
  @Aspect
  @Component
  public class LogAspects {
     
      @Pointcut("execution(public String Action.*(..))")
      private void pointCut(){};
  
      // @Before在目标方法之前切入；切入点表达式（指定在哪个方法切入）
      // 注意这个JoinPoint 参数一定在第一位
      @Before(value = "pointCut()")
      public void logStart(JoinPoint joinpoint) {
          System.out.println("logStart："+joinpoint.getSignature().getName()+"->"+ Arrays.toString(joinpoint.getArgs()));
      }
      
      // 与上面的方法效果相同
  	@Before("execution(public int MathCalculator.*(..))")
      public void logStart(JoinPoint joinpoint) {
          System.out.println("logStart："+joinpoint.getSignature().getName()+"->"+ Arrays.toString(joinpoint.getArgs()));
      }
  	
      // 目标方法之后
      @After(value ="pointCut()")
      public void logEnd(JoinPoint joinpoint) {					    					    		System.out.println("logEnd："+joinpoint.getSignature().getName()+"->"+Arrays.toString(joinpoint.getArgs()));
      }
  
  	// 目标方法成功返回后
      @AfterReturning(value ="execution(public String Action.*(..))",returning="object")
      public void logReturn(Object object) {
          System.out.println("logReturn："+object);
      }
  	
      // 目标方法出现异常后
      @AfterThrowing(value = "execution(public String Action.*(..))",throwing = "object")
      public void logException(Exception object) {
          System.out.println("logException："+object);
      }
  }
  
  ```

  

- 执行一下

  ```java
  package lksun.hello;
  
  import org.springframework.context.annotation.AnnotationConfigApplicationContext;
  
  public class Test {
      public static void main(String[] args) {
          AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Config.class);
          Action action = app.getBean(Action.class);
          String result = action.say("lksun");
          System.out.println(result);
          applicationContext.close();
      }
  }
  
  
  /*
  
      执行结果：
      
          代码正常执行：
              logStart：say->[lksun]
              :)
            logEnd：say->[lksun]
              logReturn：Hellolksun!!
              Hellolksun!!
  
          添加错误代码: 
              logStart：say->[lksun]
              :)
              logEnd>>>>>say>>>>[lksun]
              logException>>>>java.lang.ArithmeticException: / by zero
              
              Exception in thread "main" java.lang.ArithmeticException: / by zero
                  at lksun.hello.Action.say(Action.java:9)
  */
  ```
  
  

## 参考

- [Docs4dev - Spring Framework 中文文档](https://www.docs4dev.com/docs/zh/spring-framework/4.3.21.RELEASE/reference/aop.html#aop-introduction)
- [Spring AOP使用示例](https://blog.csdn.net/ysl19910806/article/details/91898875)
- [Springboot源码分析之EnableAspectJAutoProxy](https://segmentfault.com/a/1190000020158072)

