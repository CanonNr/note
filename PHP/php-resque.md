## PHP-Resque

### 准备

- 环境配置

  - Win10
  - PHP7.1
  - Redis
  - Composer

  

- 安装必须软件

  - Composer

    - 可以在官网直接下载相应exe文件；

    - Linux环境下可以通过

      ```SHELL
      $ apt-get install composer
      ```

    

  - 装PHP-Resque

    ```shell
    # 旧版本：
    composer require  "chrisboulton/php-resque 1.2"
    # 新版本：
    composer require resque/php-resque
    ```



- 介绍下目录

  在一个空目录下执行完Composer命令后会出现以下文件或目录：

  ```php
  /*
  
  |--vendor/
  	|--bin/
  		|--resque  执行队列会引用他 php-cli下执行
  	|--colinmollenhour/
  	|--composer/
  	|--psr/
  	|--resque/
  		|--php-resque	目录下有官方的DEMO和包核心文件
  	|--autoload.php 	Composer自动加载文件
  |--composer.json		
  |--composer.lock
  
  **/
  ```



- 需要编写的内容

按照消息队列的原理我们至少需要以下文件：

1. 一个入口文件用于将目标存入队列（**queue.php**）
2. 一个执行文件进行计划内的操作（**My_Job.php**）
3. 一个被守护的进程持续读取队列中的数据（**woker.php**）



### 开始

```php
# queue.php 生成测试数据存入队列中,可以直接WEB访问也可以通过cli

//导入自动加载的文件
require 'vendor/autoload.php';
//配置redis
Resque::setBackend('localhost:6379');
//生成一个随机的id 并存入数组
empty($_GET['name']) ?  $name = rand(1000,9999)  :  $name = $_GET['name'];

$args = array(
    'name' => $name,
);

//存入队列操作 并返回一个结果
if(Resque::enqueue('default', 'My_Job', $args)){
    echo "enqueue:{$name} \n";
    die();
}else{
    echo "error:{$name} \n";
    die();
}
	
```



```php
# My_Job.php  文件名无所谓,类名需要好好写  后面是需要的
class My_Job
{
    public function perform()
    {
        //在cli下输出name
        echo $this->args['name'];
        //将name写入一个txt文件中,主要用于测试
        $file = fopen('./log.txt',"a+");
        fwrite($file,$this->args['name']."\n");
        fclose($file);
    }
}
```



```php	
# worker.php   需要通过cli持续运行并守护的文件,代码很简单
require 'My_Job.php';
require './vendor/bin/resque';
```



### 操作流程

```SHELL
# 首先生成测试数据存入队列
php queue.php
# 按照之前代码成功后会返回一个name
enqueue:8185
# 此时如果你去Redis可视化管理工具或者敲命令 会在db0中看到resque相关数据

# 开启worker文件
QUEUE=* php worker.php
# 前面的QUQU是什么?
# 这是一个必填的参数,worker会根据它去执行什么任务
# 因为在一个大型应用中可能会有很多需要队列的场景,例如:短信,邮件,日志等
# 而QUEUE=* 表示了执行所有任务

#!/usr/bin/env php
[notice] Starting worker SUN:9724:*
ps: unknown option -- A
Try `ps --help' for more information.
[notice] Starting work on (Job{default} | ID: 25329c7295c0eebbf6eb38e8403f9d60 |
 My_Job | [{"name":8185}])
 
 # 讲一个笑话:PHP中notice是一种报错虽然不影响使用,在此我也以为这是一个报错,测试了一会找不到头绪,后来看了文档才发现 这是一个成功的通知!
 
 # 还可以去看一下log.txt文件有没有相应的数据
 # emmm,我实在懒得截图了
```



## 其他

**PHP-Resque** 的环境变量有：

- **QUEUE** – 这个是必要的，会决定 worker 要执行什么任务，重要的在前，例如 QUEUE=notify,mail,log 。也可以设定為 QUEUE=* 表示执行所有任务。
- **APP_INCLUDE** – 可选，加载文件用的。可以设成 APP_INCLUDE=require.php ，在 require.php 中引入所有 Job 的 Class即可。
- **COUNT** – 设定 worker 数量，预设是1 COUNT=5 。
- **REDIS_BACKEND** – 设定 Redis 的 ip, port。如果没设定，预设是连 localhost:6379 。
- **LOGGING, VERBOSE** – 设定 log， VERBOSE=1 即可。
- **VVERBOSE** – 比较详细的 log， VVERBOSE=1 debug 的时候可以开出来看。
- **INTERVAL** – worker 检查 queue 的间隔，预设是五秒 INTERVAL=5 。
- **PIDFILE** – 如果你是开单 worker，可以指定 PIDFILE 把 pid 写入，例如 PIDFILE=/var/run/resque.pid 。
- **BACKGROUND** 可以把 resque 丢到背景执行。或者使用 `php resque.php &`就可以了。

