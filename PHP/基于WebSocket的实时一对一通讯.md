# 基于WebSocket的实时一对一通讯

我自己都忘了第一次学习`WokerMan`、`WebSocket`是什么时候了但是第一次实战应该还是在今年初，当时用的急没太细看代码就按照文档直接梭哈了，现在有时间了决定还是重新走一遍当时的路所以还是一篇回忆+理论的文章，没有太多图片及步骤。

`Swoole`和`WokerMan`最大的区别可能就是前者是纯C编写PHP的一个官方扩展后者是PHP编写的，网上也出现很多两者的比较，反正我够用就行。

## 安装WokerMan



安装相比较`Swoole`简单的多，一行`Composer`语句就够了

```
composer require workerman/workerman
```



## 启动WokerMan服务

安装了包并不能直接使用需要开启相对应`server`服务。

* 举个例子原生的写法是：

  ```PHP
  <?php
  # 定义命名空间及导入扩展
  use Workerman\Worker;
  require_once __DIR__ . '/Workerman/Autoloader.php';
  
  // 注意：这里与上个例子不同，使用的是websocket协议
  $ws_worker = new Worker("websocket://0.0.0.0:2000");
  
  // 启动4个进程对外提供服务
  $ws_worker->count = 4;
  
  // 当收到客户端发来的数据后返回hello $data给客户端
  $ws_worker->onMessage = function($connection, $data)
  {
      // 向客户端发送hello $data
      $connection->send('hello ' . $data);
  };
  
  // 运行worker
  Worker::runAll();
  ```

  



命令行运行：php server.php

