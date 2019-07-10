### 开发环境

Lumen 5.7

PHP 7.2

Win10



## 开始



### 在Lumen中创建一个中间件

因为Lumen不同于Laravel，没有内置Artisan命令需要自己创建了。

1. 复制**\app\Http\Middleware\ExampleMiddleware.php**文件并将文件重命名为**AccessMiddleware.php**

2. 记得文件内的类名也要修改为**AccessMiddleware**

3. 在**/bootstrap/app.php下**添加注册代码

   ```php
   $app->routeMiddleware([
       ...
       ...
       'access' => App\Http\Middleware\AccessMiddleware::class,
   ]);
   ```

   

4. 到路由文件下添加路由组

   ```php
   $router->group(['middleware' => 'access'], function($router)
   {
   
   	$router->get('/test','TestController@Show');
   
   });
   # 截止到现在,一个普通的路由创建完成,路由名称可以根据心情修改
   ```

   

5. 回到**AccessMiddleware.php**中

   ```php
   public function handle($request, Closure $next)
   {
       header("Access-Control-Allow-Origin: *");
       header('Access-Control-Allow-Headers: X-Requested-With,X_Requested_With');
       return $next($request);
   }
   ```

   

### 相关插件

[laravel-cors](https://github.com/barryvdh/laravel-cors)

[medz/cors](https://github.com/medz/cors)

## end

