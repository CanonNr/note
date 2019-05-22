# 基于共享Session的Laravel单点登陆

摸索了一会,原理上没细研究直接写结果了。

## 多站点

单点登陆首先需要的就是多站点,可以使一样的网站也可以是有关联的.

首先就是要同一域名例如：[a.baidu.com，b.baidu.com ...]

本地可修改HOST

> 个人理解：用户登陆肯定是少不了`Cookie`的,为了安全只有同域名下浏览器才允许获取

## 共享Session

之前有讲过，就是将Session改为`Redis`驱动并解除私有

- [Laravel中将Session改为Redis驱动](.//PHP/Laravel中将Session改为Redis驱动.md)
- [Redis在Laravel中拒绝远程访问](../PHP/Redis在Laravel中拒绝远程访问.md)

## 修改配置

在根目录`.ENV`中添加`SESSION_DOMAIN=.xxx.com`

`.xxx.com`输入自己的域名，最前面那个点一定不要漏

设置这个的原因是在`config/session`中引用过这个配置

````
 'domain' => env('SESSION_DOMAIN', null)  # 154行
````

第二个参数为默认值,如果没有设置就默认为空了

## 测试

为了方(tou)便(lan)，我直接跑了两台虚拟机(完全一样的代码)，通过修改`HOST`

在无痕模式下登陆其中一个系统，另一个直接有出现了用户头像无需登陆。
