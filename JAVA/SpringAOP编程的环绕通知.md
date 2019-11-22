## 概述
- 问题：**当我们配置了环绕通知之后,切入点方法没有执行,而通知方法执行了**

- 分析： 通过对比动态代理的环绕通知代码,发现动态代理的环绕通知有明确的切入点方法调用.而我们的没有

  

- 解决：

  Spring框架为我们提供了一个接口,proceedingJoinPoint.该接口有一个方法proceed()此方法就相当于明确调用切入点方法.

  该接口可以作为环绕通知的方法参数,在程序执行时,Spring框架会为我们提供该接口的实现类供我们使用

  

- Spring中的环绕通知： 它是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式

## 代码

```java
@Aspect
public class AnnotationAudienceAround{
    //使用@Pointcut注解声明切入点表达式
    @Pointcut("execution(* com.qin.util.*.*(..))")
    public void pt1(){}

    @Around("pt1()")
    public Object aroundPringLog(ProceedingJoinPoint pjp){
        Object rtValue = null;
        try{
            // 得到方法执行所需的参数
            Object[] args = pjp.getArgs();

            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。前置");
            
            // 明确调用业务层方法（切入点方法）
            rtValue = pjp.proceed(args);

            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。后置");

            return rtValue;
        }catch (Throwable t){
            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。异常");
            throw new RuntimeException(t);
        }finally {
            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。最终");
        }
    }
}

```



## 总结

- 因为**注释**的**切入点通知**存在顺序上的错误（Before、After、AfterReturning、AfterThrowing），使用环绕通知不会出现这种问题。
- 环绕通知其实就是给了我们一个自己发挥的空间，根据逻辑定制各种通知。



## 参考

- [Spring中的环绕通知](https://blog.csdn.net/qq_35472880/article/details/83475210)

- 黑马学院