## 前言

这次可以说是手动搭建了,不使用集成工具。效果就行一台Linux下跑多个项目,但是他们使用的是同一个php环境;

先说下需要的食材：一台**Ubuntu16.4**并安装了Docker。



## 开始

Docker的核心是容器，一个完整的PHP环境至少包括以下几个容器：

* PHP
* MySQL
* Nginx、Apache

所以先从创建容器开始，我们提供**Docker命令**和**Docker-Compose**两个方法。



###  Docker命令

之前笔记写过了就不多讲了，简单说下

```shell
$ sudo docker pull [镜像名称]   拉取镜像
$ sudo docker images 	查看下载的镜像
$ sudo docker run [容器名] [镜像]... 生成容器

# 说两个之前没有用到的参数
--privileged=true 高级权利,挂载目录后容器可以访问目录以下的文件或者目录
--link使两个容器链接到一起，之间可以进行通讯，解除了对IP的依赖
```



但是已经被Docker命令折磨的不行了，所以这次没有通过命令生成容器。



### Docker-Compose

> Docker Compose是一个用来定义和运行复杂Docker应用的工具。一个使用Docker容器的应用，通常由多个容器组成。使用后就不无需要使用shell脚本来启动容器。 

