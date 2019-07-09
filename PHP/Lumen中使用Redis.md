## 1. 安装扩展

要使用redis必须安装两个扩展

```php
 composer require predis/predis
 composer require illuminate/redis
```



## 2. 引入redis支持

在目录`bootstrap/app.php`中要引入redis的扩展

```php
$app->register(Illuminate\Redis\RedisServiceProvider::class);
```



## 3. 启用redis辅助函数

> Lumen和Laravel有些不一样，默认’Facades’和’Eloquent’是没有启用的，要想像laravel中使用redis一样，要把文件`bootstrap/app.php`里的’Facades’和’Eloquent’的` $app->withFacades()` 和 `$app->withEloquent()`注释打开就好了



## 4. 配置redis服务器参数

默认系统是调用的`.env`里的redis配置文件，但是一般安装后没有这些参数，可以查看文件路径`vendor/laravel/lumen-framework/config/database.php`中查看有哪些参数需要配置，例如，我的`.env`文件需要配置

```php
REDIS_HOST=192.168.1.41
REDIS_PORT=7000
REDIS_PASSWORD=123456
```



## 5. 使用redis

首先要在使用redis的控制器内引入类。`use Illuminate\Support\Facades\Redis`
然后就可以直接使用redis函数了

```php
Redis::setex('site_name', 10, 'Lumen的redis');
return Redis::get('site_name');
```



## 6. 第二种使用redis的方法

> 使用辅助函数Cache一样可以调用redis

首先要在使用redis的控制器内引入Cache类。`Illuminate\Support\Facades\Cache` 然后就可以直接使用redis函数了

```php
Cache::store('redis')->put('site_name', 'Lumen测试', 10);
return Cache::store('redis')->get('site_name');
```



## 7. 第三种使用redis的方法



```php
# 实例化
$redis = app('redis.connection');
# 存
$redis->setex('token',300,'xxx');
# 取
$captchaCode = $redis->get('token');
```

