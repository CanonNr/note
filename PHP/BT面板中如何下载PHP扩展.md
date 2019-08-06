## BT面板中如何下载PHP扩展

BT在集成环境中用的算比较多的也自带了可以一键安装PHP扩展，但是如果遇到一些冷门或者新扩展是不能一键安装的。今天正好需要安装PHP中的Yar扩展记录一下。

## 环境

- CentOS 7.6
- BT面板6.9.8
- PHP7.1

## 准备

- [Yar官方GIT](https://github.com/laruence/yar)

## 开始

> ​	因为服务器中没有连接外网所以没有使用Git命令直接克隆而是本机下载后FTP到服务器中

```shell
# 按照官方安装步骤是：
$ cd /yar
$ /path/to/phpize
$ ./configure --with-php-config=/path/to/php-config/
$ make && make install

# 当然我们并不能这么做，需要修改下路径
# phpize 和 php-config的路径
# 直接揭晓答案他们分别在：
# /www/server/php/71/bin/phpize
# /www/server/php/71/bin/php-config
# 得到新的命令
$ cd /yar
$ /www/server/php/71/bin/phpize
$ ./configure --with-php-config=/www/server/php/71/bin/php-config
$ make && make install 

# 执行完毕后会返回一堆东西 如下:
....
Build complete.
Don't forget to run 'make test'.

Installing shared extensions:     /www/server/php/71/lib/php/extensions/no-debug-non-zts-20160303/

# 其他不重要主要是那个路径
# 并且在这个路径下会多出一个yar.so的文件


# 最后一步,到php.ini文件中添加新扩展
$ cd /www/server/php/71/etc/php.ini
# 咋最后面加上
...
[redis]
extension = /www/server/php/71/lib/php/extensions/no-debug-non-zts-20160303/redis.so
[yar]
extension = /www/server/php/71/lib/php/extensions/no-debug-non-zts-20160303/yar.so

# 重启php
$ /etc/init.d/php-fpm-71 restart

# 查看是否已经有了新扩展
$ php -m 
# 或者通过phpinfo();

```

md