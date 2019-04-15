#	Swoole

* [Swoole官网](https://www.swoole.com/)
* [GitHub](https://github.com/swoole/swoole-src)
* [官方文档](https://wiki.swoole.com/)

## ~~Win环境下的Swoole安装~~

> Win环境下不支持，需要下载[cgywin](https://www.cnblogs.com/itsuibi/p/8995137.html)，太麻烦了溜了溜了直接上Linux

## 	开始

- Deepin 15.9 Desktop
- PHP 7.1
- Nginx

### 起步第一个坑

  > Fatal error: swoole_server::__construct(): swoole_server only can be used in PHP CLI mode. in /www/..../index.php on line 3

  必须通过PHP CLI ，WEB环境是不行的

  ```shell
  php /www/..../index.php
  
  #成功执行Swoole服务器程序后，如果你的代码中没有任何echo语句，屏幕不会有任何输出，但实际上底层已经在监听网络端口，等待客户端发起连接。可使用相应的客户端工具和程序连接到服务器，进行测试。
  ```

 
