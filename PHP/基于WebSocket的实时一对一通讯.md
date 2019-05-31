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



* 原生的写法是：

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

  

* Laravel中运行服务

  ```PHP
  # 在框架下原生写法当然也可以，但是有更好的命令行还是使用命令
  # 创建一个Atisan命令
  # 不写了 文档都有
  # 把指定Server代码写在 Command文件的handle下就好了
  ```

  



## 开始

首先讲下需求：A控制器做判断，为`true`时Show页面产生相应动作。



### 代码

```php
# 服务端
global $worker;
$worker = new Worker('websocket://127.0.0.1:1234');
// 这里进程数必须设置为1
$worker->count = 1;
// worker进程启动后建立一个内部通讯端口
$worker->onWorkerStart = function($worker)
{
    // 开启一个内部端口，方便内部系统推送数据，Text协议格式 文本+换行符
    $inner_text_worker = new Worker('Text://127.0.0.1:5678');
    $inner_text_worker->onMessage = function($connection, $buffer)
    {
        global $worker;
        // $data数组格式，里面有uid，表示向那个uid的页面推送数据
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // 通过workerman，向uid的页面推送数据
        $ret = sendMessageByUid($uid, $buffer);
        // 返回推送结果
        $connection->send($ret ? 'ok' : 'fail');
    };
    $inner_text_worker->listen();
};
// 新增加一个属性，用来保存uid到connection的映射
$worker->uidConnections = array();
// 当有客户端发来消息时执行的回调函数
$worker->onMessage = function($connection, $data)use($worker)
{
    // 判断当前客户端是否已经验证,既是否设置了uid
    if(!isset($connection->uid))
    {
        // 没验证的话把第一个包当做uid（这里为了方便演示，没做真正的验证）
        $connection->uid = $data;
        /* 保存uid到connection的映射，这样可以方便的通过uid查找connection，
                 * 实现针对特定uid推送数据
                 */
        $worker->uidConnections[$connection->uid] = $connection;
        return;
    }
};

// 当有客户端连接断开时
$worker->onClose = function($connection)use($worker)
{
    global $worker;
    if(isset($connection->uid))
    {
        // 连接断开时删除映射
        unset($worker->uidConnections[$connection->uid]);
    }
};

// 向所有验证的用户推送数据
function broadcast($message)
{
    global $worker;
    foreach($worker->uidConnections as $connection)
    {
        $connection->send($message);
    }
}

// 针对uid推送数据
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// 运行所有的worker
Worker::runAll();
```



```php
# A控制器
if(true){
	$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
	// 发送数据，注意5678端口是Text协议的端口，Text协议需要在数据末尾加上换行符 $data为要发送的数据
	fwrite($client, json_encode($data)."\n"); 
	// 读取推送结果
	return fread($client, 8192);  
}
```



```html
<!--Show页面--> 
<script>
    var ws = new WebSocket('127.0.0.1');
    ws.onopen = function(){
        ws.send(uid); //像后端发送一个ID 算是对当前用户的注册
    };
  
    //接收数据时执行
    ws.onmessage = function(e){
        //动作写在这里
        var data =  JSON.parse(e.data);  //传送的数据
    }
</script>
```

