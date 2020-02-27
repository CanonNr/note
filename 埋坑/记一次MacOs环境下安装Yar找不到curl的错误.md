## 前言

之前在Centos7下安装的很顺利这次无论使用**编译安装**还是**pcel**都会提示一个缺少**curl**

**configure: error: Please reinstall the libcurl distribution - easy.h should be in <curl-dir>/include/curl**

但是通过命令行执行都没问题：

```shell
which curl
/usr/bin/curl

curl baidu.com
<html>
<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
</html>
```



## 分析

 mac和linux目录结构不同没有**/usr/include**目录

解决方式：

- 修改源码`conmfig.m4`下修改curl的执行目录

  // TODO 理论可行但是尝试并未成功,以后有机会再埋坑

- 编译过程中手动选择curl路径

  ```shell
  cd yar
  phpize
  ./configure --with-php-config=/Applications/MAMP/bin/php-config --with-curl=/usr/local/opt/curl
  make && make install
  ```

  

