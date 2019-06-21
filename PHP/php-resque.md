# php-resque



## 消息队列



### 定义



### 优势



### 应用场景



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



### 看下效果

