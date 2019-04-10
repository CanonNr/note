#	用户ID设计

* UUID
  * hashids

##	UUID

​	UUID 是 通用唯一识别码（Universally Unique Identifier）的缩写，是一种软件建构的标准，亦为开放软件基金会组织在分布式计算环境领域的一部分。

​	其目的，是让分布式系统中的所有元素，都能有唯一的辨识信息，而不需要通过中央控制端来做辨识信息的指定。如此一来，每个人都可以创建不与其它人冲突的UUID。在这样的情况下，就不需考虑数据库创建时的名称重复问题。

###	组成

​	UUID是指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。通常平台会提供生成的API。按照开放软件基金会(OSF)制定的标准计算，用到了以太网卡地址、纳秒级时间、芯片ID码和随机数。

* 当前日期和时间，UUID的第一个部分与时间有关，如果你在生成一个UUID之后，过几秒又生成一个UUID，则第一个部分不同，其余相同。
* 时钟序列。
* 全局唯一的IEEE机器识别号，如果有网卡，从网卡MAC地址获得，没有网卡以其他方式获得。

###  代码生成



```php
function uuid($prefix = '')
{
    $chars = md5(uniqid(mt_rand(), true));
    $uuid  = substr($chars,0,8) . '-';
    $uuid .= substr($chars,8,4) . '-';
    $uuid .= substr($chars,12,4) . '-';
    $uuid .= substr($chars,16,4) . '-';
    $uuid .= substr($chars,20,12);
    return $prefix . $uuid;
} 
```



###	参考

[UUID 百度百科](https://baike.baidu.com/item/UUID/5921266?fr=aladdin)





------



##	hashids

> A small PHP class to generate YouTube-like ids from numbers.
>
> 这是一个轻量级库用来生成混淆字符串
>
> Hashids是一个能利用整数生成出短小、唯一、非连续标识符的类库，它支持包含php在内的好多好多（真的好多）种语言。
>
> Hashids支持通过生成出来的标识符进行解码为原数字，还支持加盐加密，不会因为大家都用这个类库就被猜到真实ID。

### 使用

```php
$hashids = new Hashids\Hashids();//实例化hashids

$id = $hashids->encrypt(1, 2, 3);//加密多个值,也可以加密单个值

$numbers = $hashids->decrypt($id);//解密哈希值

//自定义SALT值用法如下;
//Hashids类的构造函数有三个参数分别是：
$salt = '', $min_hash_length = 0, $alphabet = ''
$salt					//是自定义的SALT值，最好大于6位小于32位。
$min_hash_length		 //是哈希值最小长度,最少7位。
$alphabet				//是加密生成的字符串来源默认是abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890，可以自己定义，但要多于16个。


$hashids = new Hashids\Hashids('this is my salt', 8, 'abcdefghij1234567890'); //实例化

$id = $hashids->encrypt(1, 2, 3); //加密

$numbers = $hashids->decrypt($id); //解密

//加盐加密
$hashids = new Hashids\Hashids('我是盐');
// 编码
$hashID = $hashids->encode($id);
// 解码
$decodeResult = $hashids->decode($hashID);
```

### 扩展

hashids生成的混淆字符串是随机性的。

由于它本身避免了以下字符串相邻，所以大部分英文的脏话是不会出现的。

> ​	c, C, s, S, f, F, h, H, u, U, i, I, t, T

### 参考

[hashids官网](https://hashids.org/php/)

[博客园](https://www.cnblogs.com/lancode/p/3964460.html)

[宇润博客](http://blog.yurunsoft.com/a/Hashids.html)

[官方github](https://github.com/ivanakimov/hashids.php)