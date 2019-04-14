##	错误一

> Warning：
>
> Your requirements could not be resolved to an installable set of packages.

> 解决方法：
>
> composer install --ignore-platform-reqs 
>
> composer update --ignore-platform-reqs

php版本不符,打开composer.json发现代码只要求大于7.0.0可仍然报这个错误,不用太较真执行如下代码之后在运行之前的代码就ok



## 错误二

> Warning: 
>
> This development build of composer is over 60 days old.

> ​	/usr/local/bin/composer  self-update
>
> 若前半句找不到文件则通过 whereis composer