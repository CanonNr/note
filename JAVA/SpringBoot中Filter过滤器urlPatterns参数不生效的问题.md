## 起因

在前面写[登录功能](./SpringBoot中使用Filter过滤器实现登录功能.md)的时候用到了Filter去过滤没有登录状态的请求，我是过滤了所有路径（也就是`urlPatterns = "/"`）然后手动跳过了其他的一些路径。

随即又想到一个需求：当已经登录后进入`login`页面自动跳到首页，按照之前的方法在`/Fileter`下又创建了一个过滤器仅过滤`urlPatterns = "/login"`的请求。

这是学习JAVA以来独自排查的问题，详细写一下算有个纪念意义。

### 代码

```JAVA
@Component
@WebFilter(filterName = "loginFilter", urlPatterns = "/login")
public class LoginFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        HttpSession session = request.getSession();
        Object user = session.getAttribute("user");
        if (user == null){
            filterChain.doFilter(servletRequest, servletResponse);
        }else{
            response.sendRedirect("/");
        }
    }

    @Override
    public void destroy() {
    }

}

```



### 错误

登录前一切都正常，登录成功后有一个“登录成功。。。 正在跳转的提示” 然后跳转到首页的时候浏览器就显示**重定向次数过多的错误**。原因呢就在过滤器中最后一个判断了：`如果Session中没有用户信息则正常显示，反之跳转到首页`就是在此时反复跳转到首页所以报了重定向的错误。

然后在`doFilter`中输出`request.getRequestURI()`重新走一遍发现，控制台有`/`输出，按道理这个过滤器是不执行`/login`之外请求的，过滤器执行了只是没有按照配置在指定的路由执行所以判定是`urlPatterns`没有生效



### 排查思路

- 遇到错误常用的伎俩：
  - 看报错
    - 虽然效果不符合预期但是控制台和浏览器并没有错误 

      

  - 看日志

    都是Info级别的日志也很正常没什么错误，所以决定降低一下日志的显示等级：

    ```yml
    # YML配置中
    logging:
      level:
        root: debug
    ```
    

- 然后在找到了这么一段：

  ```XML
  Mapping filters:
      filterRegistrationBean urls=[/*] order=-2147483598, 
      characterEncodingFilter urls=[/*] order=-2147483648, 
      formContentFilter urls=[/*] order=-9900, 
      requestContextFilter urls=[/*] order=-105, 
      authFilter urls=[/*] order=1, 
      loginFilter urls=[/*] order=2147483647
  ```

  
    其中有`authFilter`和`loginFilter`都存在但是`urls`不对。
  
    做了个小测试：我将`filterName`改为`loginasdasFilter`重新查看日志依然还是`loginFilter`,所以证明`@WebFilter`并没有生效



- 然后查到`@ServletComponentScan`

  其作用是：在入口的`Application`类上使用`@ServletComponentScan`注解后，`Servlet`、`Filter`、`Listener`可以直接通过`@WebServlet`、`@WebFilter`、`@WebListener`**注解自动注册，无需其他代码**。

  所以就可以删掉`@Component`了

  此时的日志：

  ```XML
  filterRegistrationBean urls=[/*] order=-2147483598, 
  AdminAuthFilter urls=[/*] order=2147483647, 
  loginFilter urls=[/login] order=2147483647, 
  characterEncodingFilter urls=[/*] order=-2147483648, 
  formContentFilter urls=[/*] order=-9900, 
  requestContextFilter urls=[/*] order=-105
  ```

  