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

按照消息队列的原理，



### 开始