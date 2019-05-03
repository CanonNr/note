# ThinkPHP6.0入门

碰巧前两天在论坛看到TP6.0发布的新闻，之前很长一段时间使用的都是3.2也算有点感情。

## 新特性

ThinkPHP`6.0`在`5.1`的基础上对底层架构做了进一步的精简和统一，引入了一些新特性，并提升版本要求。

官网给了很多新特性，列出几条最关心或最重要的：

* 采用`PHP7`强类型（严格模式）
* SESSION机制改进（官方解释为方便Swoole）
* 对Swoole以及协程支持改进
* 原生多应用支持

## 安装

### 要求

> PHP >= 7.1.0

> `6.0`版本开始，必须通过`Composer`方式安装和更新。

### 安装

看到``必须通过Composer方式安装和更新``就知道TP这次是铁了心的使用Composer了，所以就按照官方要求进行下载。

> 其实如果不想使用Composer也可以搜到一些第三方的压缩包、Git的下载方式。

### 第一次安装

如果你是第一次安装的话，在命令行下面，切换到你的WEB根目录下面并执行下面的命令：

```
composer create-project topthink/think tp
```

### 更新

如果你之前已经安装过，那么切换到你的**应用根目录**下面，然后执行下面的命令进行更新：

```
composer update topthink/framework
```



## 开始

> 首先我的环境是Linux（Deepin桌面版）+ Nginx 1.15 + MySQL 5.5 + PHP7.1 

### Nginx规则

除了首页外出现404错误，需要修改配置文件

````
location ~/ {
	if (!-e $request_filename) {
		rewrite ^(.*)$ /index.php?s=/$1 last;
		break;
	}
}
````



旧URL：

```
http://serverName/index.php/模块/控制器/操作/[参数名/参数值...]
```

设置后：

```
http://serverName/模块/控制器/操作/[参数名/参数值...]
```

例如设置了三个路由：

```php
Route::get('hello/:name', 'index/hello');
Route::get('test', 'Test/index');
Route::get('show', 'Test/show');
```

就可分访问

* 127.0.0.1:8802/hello/100
* 127.0.0.1:8802/shows
* 127.0.0.1:8802/test



## Think 命令

和Laravel的相似，尝试了一下创建控制器的命令

```
# 切换至项目根目录
php think make:controller User

# 得到结果
Controller:app\controller\User created successfully.

```



然后就多了一个``User.php``的控制器

> 其他的命令也就类似了，不一一尝试了贴上文档需要使用时看。
>
> [命令行](https://www.kancloud.cn/manual/thinkphp6_0/1037640)