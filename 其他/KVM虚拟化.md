#	KVM虚拟化



## 前言

公司虚拟化环境由**VMware ESXI**转为**KVM**,所以学习记录一下。

至于为什么不使用当下最火的Docker.....原因很简单,领导指名道姓要用KVM。

搭好并实际使用后再写一个**Docker**、**VMware ESXI**、**KVM**三者的对比吧。

###	ESXI 使用感受

VMware一直算是虚拟化的龙头了使用了半年之久的vSphere感觉还是不错的,给客户部署系统需要经常导入导出OVF也很简单易操作。

当然也有缺点或者叫需要改进的地方：

- 虽然导入导出操作简单,但是速度慢!网络没升级千兆之前一个操作经常要半个小时。
- OVF镜像过于大,ubuntu都在8G左右,不知道KVM能不能得到改善
- 中间有一次莫名其妙管理密码就不对了,只能抹掉全盘重新安装(破解问题?)

### KVM

使用了一天的KVM,也算是勉强入了门，说说看法：

* 开源，免费
* 原生操作比 ESXI要复杂一些，要试着找找图形化的软件了。
* 创建镜像直接生成了一个30G的文件，意味着导入导出也要30G？

## 虚拟化

### 什么是虚拟化

​	虚拟化，是指通过虚拟化技术将一台计算机虚拟为多台逻辑计算机。在一台计算机上同时运行多个逻辑计算机，每个逻辑计算机可运行不同的操作系统，并且应用程序都可以在相互独立的空间内运行而互不影响，从而显著提高计算机的工作效率。

### 什么是虚拟化技术

​	虚拟化技术是一套解决方案。完整的情况需要CPU、主板芯片组、BIOS和软件的支持，例如VMM软件或者某些操作系统本身。即使只是CPU支持虚拟化技术，在配合VMM的软件情况下，也会比完全不支持虚拟化技术的系统有更好的性能。

### 虚拟化类型

#### 全虚拟化（Full Virtualization)
​	全虚拟化也成为原始虚拟化技术，该模型使用虚拟机协调guest操作系统和原始硬件，VMM在guest操作系统和裸硬件之间用于工作协调，一些受保护指令必须由Hypervisor（虚拟机管理程序）来捕获处理。全虚拟化的运行速度要快于硬件模拟，但是性能方面不如裸机。

#### 半虚拟化（Para Virtualization）
​	半虚拟化是另一种类似于全虚拟化的技术，它使用Hypervisor分享存取底层的硬件，但是它的guest操作系统集成了虚拟化方面的代码。该方法无需重新编译或引起陷阱，因为操作系统自身能够与虚拟进程进行很好的协作。半虚拟化需要guest操作系统做一些修改，使guest操作系统意识到自己是处于虚拟化环境的，但是半虚拟化提供了与原操作系统相近的性能。



## 开始安装

### 准备一台宿主机

* 如果基于虚拟化的Linux下安装KVM需要在设置中开启各种开关(百度可以搜到好多 不写了)
* 如果是基于物理机下安装：
  * 需要判断CPU是否支持虚拟化。
  * 主板Bios中是否开启虚拟化。

由于我懒得找开启虚拟化开关以及电脑内存不够防止走弯路等，我还是选择了找一台物理机。

> 宿主机配置：
>
> I5+8G+1T  
>
> CentOS  7.6.1810 x86_64

因为这台主机之前是装过vSphere的我就跳过了是否支持虚拟化的检测。

安装CentOS 时我选择格式化了全盘。

### 安装

```shell
 $ yum -y install qemu-kvm libvirt virt-install bridge-utils virt-viewer
 $ lsmod | grep kvm 
# kvm_intel             183621  3 
# kvm                   586948  1 kvm_intel
# irqbypass              13503  3 kvm
$ systemctl start libvirtd 
$ systemctl enable libvirtd 
```

### 为虚拟机配置桥连网络

```shell
# 首先ifconfig获取网卡信息
$ ifconfig

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.223  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::1766:473e:de5d:ab6b  prefixlen 64  scopeid 0x20<link>
        ether 52:54:00:32:5b:6c  txqueuelen 1000  (Ethernet)
        RX packets 138851  bytes 285157949 (271.9 MiB)
        RX errors 0  dropped 6  overruns 0  frame 0
        TX packets 105048  bytes 11291806 (10.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

$ cp ifcfg-eth0 ifcfg-br0    # 拷贝一份当前的网卡配置文件
$ vim ifcfg-eth0  			# 在尾行添加 BRIDGE=br0
$ vim ifcfg-br0  			# 修改文件内容如下

TYPE=Bridge                  # 修改类型
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=br0					# 修改
DEVICE=br0					# 修改
ONBOOT=yes					# 修改

$ systemctl restart network  # 重启网卡服务

# 重启过后之前eth0的网卡IP就到了br0上了

br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.77.130  netmask 255.255.255.0  broadcast 192.168.77.255
        inet6 fe80::20c:29ff:fef1:912c  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:f1:91:2c  txqueuelen 0  (Ethernet)
        RX packets 51  bytes 8341 (8.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 27  bytes 2710 (2.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 00:0c:29:f1:91:2c  txqueuelen 1000  (Ethernet)
        RX packets 147615  bytes 168580073 (160.7 MiB)
        RX errors 0  dropped 8  overruns 0  frame 0
        TX packets 45008  bytes 3866579 (3.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 2459  bytes 1125227 (1.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2459  bytes 1125227 (1.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```



### 终端模拟器

对于文字式的安装系统我是拒绝的,所以之前就安装了``virt-viewer``工具。接下来在WIN下安装一个``Xmanager 6``配合``Xshell 6``这样安装虚拟机的时候遇到可视界面就可以模拟出一块屏幕。

> 一个插曲：
>
> ​	后续使用Xmanager 时会出现键盘按一下终端出现两个字符的情况。
>
> 解决方法：
>
> 1. 运行**Xconfig**，找到**Default Profile**，右键**属性** -> **高级** -> 找到**XKEYBOARD** 去掉 **√**
>
> 2. 输入法问题，切换至英文状态下，不要使用任何QQ、搜狐、百度输入法。
>
>    

### 创建虚拟机

* 首先创建一个目录，用于存放系统镜像。我创建的目录是 **/home/vm1/**

- 下载一个Linux的镜像。我继续使用了宿主的系统CentOS 7.6，并放置在了宿主机下 **/media/CentOS-7-x86_64-DVD-1810.iso**

  

  ```shell
  $ virt-install --name test --memory 1024 --disk /home/vm1/test.img,size=30 --location=/media/CentOS-7-x86_64-DVD-1810.iso
  ```

  * **install**命令参数有很多后续再具体写，先写几个必填的
  * **--name** 顾名思义，虚拟机的名字
  * **--memory** 内存大小（单位MB）
  * **--disk** 用于存放镜像的位置 ， **size** 是硬盘空间 （单位是GB）
  * **--location** 是系统安装文件位置，我使用的是本地文件，也可以使用**http** **ftp**等

  

  ​	执行了上面命令后就开始安装了，我使用的是**Xmanager + Xshell**，所以自动就弹出了终端模拟器，然后就是熟悉的可视化的安装界面了。

  ​	如果没有安装就会出现文字版的安装流程，我尝试了看着就头疼。。。

  

### 管理/操作 虚拟机

安装了虚拟机后会自动在**Xmanager**下运行该虚拟机，但是如果不小心关闭了或者退出了怎么办。

现在就涉及到 **virtsh** 命令了（还是只介绍部分，更多命令还需要细细研究）

```shell
$ virsh list			# 查看运行的虚拟机	
# Id    Name         State
# ---------------------------
# 15    test         running

$ virsh list --all  	# 查看所有的虚拟机
# Id    Name         State
# ---------------------------
# 15    test         running
# 16    elk          shut off
# 17    first_os     shut off

$ virt-viewer test 		# 通过模拟器进入控制台
	# 通过命令行进入,不需要其他工具(但是实测有问题,需要修改一些配置,弄好再说)
```

* 通过ssh进入虚拟机

  ```shell
  $ virsh console test
  ```

  新创建的虚拟机直接执行该命令可能会没反应不出现光标，账号密码什么的。

  > 解决方法  ：
  >
  > CentOS  下 虚拟机中直接执行
  >
  > ``*grubby --update-kernel=ALL --args=**"console=ttyS0"*``
  >
  > 然后重启虚拟机

  

  

  ```shell
  # 效果:
  $ virsh console test
  Connected to domain test14
  
  CentOS Linux 7 (Core)
  Kernel 3.10.0-957.el7.x86_64 on an x86_64
  localhost login: 
  
  ```

  



## 参考

1. [OpenStack、KVM、VMWare和Docker的参考对比](https://www.cnblogs.com/tongxiaoda/p/8559259.html)
2. [CentOS 7 安装KVM，并创建虚拟机](https://blog.csdn.net/wh211212/article/details/54141412)
3. [KVM - virsh 常用命令](https://www.cnblogs.com/lin1/p/5776280.html)
4. [KVM 通过virsh console连入虚拟机](https://blog.csdn.net/lemontree1945/article/details/80461037)
5. [网卡桥连配置](https://blog.51cto.com/zero01/2083896)
