# Laravel+JWT的用户系统.md

* Laravel 5.5
* jwt-auth 0.5.*
* PHP 7.1
* Apache

对[JWT](./JWT.md)的概念之前就有了初步的了解,这次算是真正意义上的一次实践。

因为是先操作后写的文章就没有图片了。



## 开始

### 首先在laravel中安装`jwt-auth`的包

```
composer require tymon/jwt-auth
```

### 添加服务(Laravel5.4之后的就不需要了)

```php
'providers' => [
    ...
    Tymon\JWTAuth\Providers\LaravelServiceProvider::class,
]
# 此处注意，国内很多博客论坛都有jwt-auth教程，可能因为时间关系多数文章命令与官方命令不同易出现问题
# 看了包的目录结构就会发现还有一个LumenServiceProvider::class
# 修改类名Lumen框架同样适用,步骤不同详情还是看文档吧
```

### 生成配置文件

```
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```

 这条命令后会生成一个 `config/jwt.php`的配置文件

### 生成秘钥

```
php artisan jwt:secret
```

`.ENV`文件的最后会多出一个`JWT_SECRET=xxxxxxxx`,作为JWT加密的🔑，非特殊情況无需改动。



## 配置

### 首先要修改我们的`User`的`Model`

```php
# 添加命名空间
use Tymon\JWTAuth\Contracts\JWTSubject;

# 新增了两个方法

/**
* Get the identifier that will be stored in the subject claim of the JWT.
*
* @return mixed
*/
public function getJWTIdentifier()
{
    return $this->getKey();
}

/**
* Return a key value array, containing any custom claims to be added to the JWT.
*
* @return array
*/
public function getJWTCustomClaims()
{
    return [];
}
```



### 修改`config/auth.php`文件

```php
'defaults' => [
    'guard' => 'api',
    'passwords' => 'users',
],

...

'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

将`API`的认证驱动改为`JWT`

然后我们就大功告成了。



## 实践

### 制作一个DEMO

官网给出了一个小demo，我直接拿来测试了。

```php
# 制作一个控制器
# php artisan make:controller AuthController


# 添加路由
Route::group([
    'middleware' => 'api',
], function ($router) {
    Route::post('login', 'AuthController@login');
    Route::post('logout', 'AuthController@logout');
    Route::post('refresh', 'AuthController@refresh');
    Route::post('me', 'AuthController@me');
});


# 修改代码
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Auth;
use App\Http\Controllers\Controller;

class AuthController extends Controller
{
    /**
     * Create a new AuthController instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth:api', ['except' => ['login']]);
    }

    /**
     * Get a JWT via given credentials.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function login()
    {
        # 这是一个登陆方法,下方的email+password就是认证的字段,根据实际情况填写就好 我使用的是name
        $credentials = request(['email', 'password']);

        if (! $token = auth()->attempt($credentials)) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        return $this->respondWithToken($token);
    }

    /**
     * Get the authenticated User.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function me()
    {
        # 此方法可用来登陆后返回 用户数据
        return response()->json(auth()->user());
    }

    /**
     * Log the user out (Invalidate the token).
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function logout()
    {
        # 显然易见 就是很普通的退出并返回一个message
        auth()->logout();

        return response()->json(['message' => 'Successfully logged out']);
    }

    /**
     * Refresh a token.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function refresh()
    {
        # 刷新token
        return $this->respondWithToken(auth()->refresh());
    }

    /**
     * Get the token array structure.
     *
     * @param  string $token
     *
     * @return \Illuminate\Http\JsonResponse
     */
    protected function respondWithToken($token)
    {
        # 获取token的一些信息
        # expires_in 是token过期的时间
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => auth()->factory()->getTTL() * 60
        ]);
    }
}
```



### 结果

访问`http://XXX.com/api/login`

![1558508709064](../static/1558508709064.png)

```json
{
"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC8xMjcuMC4wLjE6ODA5OVwvYXBpXC9sb2dpbiIsImlhdCI6MTU1ODUwODYzMywiZXhwIjoxNTU4NTEyMjMzLCJuYmYiOjE1NTg1MDg2MzMsImp0aSI6Ikh3SnlnOTA0TUh1Qnl3d00iLCJzdWIiOjEsInBydiI6Ijg3ZTBhZjFlZjlmZDE1ODEyZmRlYzk3MTUzYTE0ZTBiMDQ3NTQ2YWEifQ.UhVdUbZy2NXSHXRON1GN47hKAz-nu85auay_CH_koBI",
"token_type": "bearer",
"expires_in": 3600
}
```

这个`access_token`就是我们需要的

**请求头：**`Authorization: Bearer eyJhbGciOiJIUzI1NiI...`

### token使用

按照以上步骤将`JWT`应用在实际项目后发现好像系统的登录功能都异常了，莫慌只是没有将重要的`token`放在指定位置

#### 方法一

登录成功后将返回的token存储起来，请求的`URL`后面加上`token`

例如：``http://XXX.com/me?token=eyJhbGciOiJIUzI1NiI``

![1558510626849](../static/1558510626849.png)

发现获取到了用户的信息：有用户名、id、头像等

如果失败可能是token过期了



#### 方法二

设置请求头

**请求头：**`Authorization: Bearer eyJhbGciOiJIUzI1NiI...`

![1558508856864](../static/1558508856864.png)





## MVC下使用JWT

前面所述的都是在`Restful Api`下的项目，其实传统的前后端不分离`MVC`模式下也是可以的只需要稍作修改。

### 配置

```php
# 前面有修改过这个配置，这次将WEB下的driver 同样改为jwt就好了

    'guards' => [
        'web' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],
```



### 修改登录模块

```php
# 根据demo得到方法
$token = auth()->attempt($credentials)；  
# 或  
$user = User::first()；
$token = auth()->login($user);
# $token就是我们需要的jwt
```



### token使用

和之前是一样的两种方法：修改`url`或者修改`Header`

* 方法一是完全一样的，访问``http://XXX.com/me?token=eyJhbGciOiJIUzI1NiI``

  就会发现系统已经有了登录状态

* 方法二也是设置`Header`，就是位置不太一样了。

  需要给控制器写一个中间件

  ````PHP
  <?php
  
  namespace App\Http\Middleware;
  class SystemMiddleware
  {
      /**
       * Handle an incoming request.
       *
       * @param  \Illuminate\Http\Request $request
       * @param  \Closure $next
       * @return mixed
       */
      public function handle($request, Closure $next)
      {
          //jwt header
          $request->headers->set('Authorization', 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC8xMjcuMC4wLjE6ODA5OVwvYXBpXC9sb2dpbiIsImlhdCI6MTU1ODUxMDQzOCwiZXhwIjoxNTU4NTE0MDM4LCJuYmYiOjE1NTg1MTA0MzgsImp0aSI6IjE5VnBaYkh5SktEWmZjWW0iLCJzdWIiOjEsInBydiI6Ijg3ZTBhZjFlZjlmZDE1ODEyZmRlYzk3MTUzYTE0ZTBiMDQ3NTQ2YWEifQ.mgJTe_VDCPI02R7iX4vhdHAZZeDrpoEkCRGOlmKjQII');
          
      	return $next($request);
      }
  }
  ````

  

  因为只是demo没有将token写活。

  

  ```PHP
  # 要注意的是设置`header`找到数个办法这是唯一成功的，不知道为什么既然成功了就这么写了。
  # 其他的方法还有:
  header('Authorization:Bearer .....');
  return $next($request)->header('Authorization:Bearer .....');
  return $next($request)->header('Authorization','Bearer .....');
  
  ...
  ```

  

