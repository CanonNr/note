##	错误一

> Warning：
>
> Your requirements could not be resolved to an installable set of packages.

> 解决方法：
>
> composer install --ignore-platform-reqs 
>
> composer update --ignore-platform-reqs

> php版本不符,打开composer.json发现代码只要求大于7.0.0可仍然报这个错误,不用太较真执行如下代码之后在运行之前的代码就ok



## 错误二

> Warning: 
>
> This development build of composer is over 60 days old.

> s/usr/local/bin/composer  self-update
>
> 若前半句找不到文件则通过 whereis composer

## 错误三

 >Waraing：
 >
 >[Symfony\Component\Process\Exception\RuntimeException]                                   
 >
 >The Process class relies on proc_open, which is not available on your PHP installation.  

> 解决方法：
>
> 打开php.ini，并搜索disable_functions指令
>
> disable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server
>
> 删除proc_open。