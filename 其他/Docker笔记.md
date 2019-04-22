# Docker



##	基本信息

​	Docker 是一个开源的**应用容器引擎**，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现**虚拟化**。容器是完全使用沙箱机制，相互之间不会有任何接口。 



##	开始

环境为Ubuntu 16.04.0 LTS , 云服务器

### 安装

* **条件**

  * Docker目前只能在64位CPU架构的计算机上运行（目前只能是x86_64 、amd64）
  * Linux 3.8 或 更高版本的内核(Ubuntu 12.04 LTS 及之后的 64位版本)
  * 内核必须支持并开启cgroup和命名空间(banespace)功能

* **开始安装**

  

  ```shell
  # 通过Docker源安装最新版本
  $ sudo apt-get update
  $ sudo apt-get install -y apt-transport-https
  $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
  $ sudo bash -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
  $ sudo apt-get update
  $ sudo apt-get install -y lxc-docker
  ```

  

  **但是本菜鸡的我觉得太麻烦了,选择了官方的shell脚本一键安装**

  

  ```shell
  $ sudo apt-get update
  $ sudo apt-get install -y curl
  # 如果已经安装curl工具可忽略以上两步
  $ curl -s https://get.docker.com | sudo sh
  # 安装结束后终端输入 docker 后出现一堆参数 help什么的就代表成功了。
  ```

* 启动Docker服务

   ```shell
  # 启动服务
  $ sudo service docker start
  # 停止服务
  $ sudo service docker stop
  # 重启服务
  $ sudo service docker restart
  ```

## 关键字

### 镜像 Image

* 可以理解成模板
* 一个镜像(模板)可以生产处很多docker容器
* docker的模板很简单,可以直接在官方的镜像仓库拉取,各大厂商也上传了官方的docker镜像。

### 容器 Command

* 我们所说的运行Docker其实就是运行容器，也是Docker的核心。
* 容器可以理解为是由镜像产出依附在Linux下的独立系统(包括了root权限、各种端口、文件等，但是阉割版)。
* 容器与其母体互不影响，容器间也是相互隔离的，保证系统的正常运行。
* 容器可以被启动、关闭、停止、删除、上传至Docker Hub等操作。

### 仓库 Docker Hub

* 仓库就是存放了好多镜像的地方
* 仓库 ！= 仓库注册服务器，仓库注册服务器上存放了多个仓库，每个仓库中又包含了多个镜像，每个镜像有着不同的标签。
* 仓库分为公开仓库（Public）和私有仓库（Private），与GitHub相似。
* 官方的仓库是[Docker Hub](https://hub.docker.com/)，也可以自行在内网搭建私有仓库。



## 基本命令

* 获取镜像

  ```shell
  $ docker pull [OPTIONS] Name[:TAG]
  #例如：获取Ubuntu 16.04版本
  $ docker pull ubuntu:16.04
  ```

* 查看已存在镜像列表

  ```
  $ docker images
  ```

* 删除镜像

  ```shell
  $ docker rmi [OPTIONS] IMAGE [IMAGE...]
  # 强制删除ubuntu:16.04镜像
  $ docker rmi -f ubuntu:16.04
  # 删除无名镜像(悬浮镜像)
  $ docker image prune -a -f
  # 在使用的镜像无法删除
  ```

* 创建并运行一个新容器

  ```shell
  $ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
  # 创建一个Ubuntu16.04的容器
  $ docker run -it --name xxx -p 80:80 	ubuntu:16.04 /bash/bash
  # -t 表示返回一个终端
  # -i 表示打开容器的标准输入,使用这个命令可以得到一个容器的 shell 终端
  # -p 表示映射主机80端口到容器80端口
  # --name 表示容器的名称
  ```

* 查看容器列表

  ```shell
  # 查看运行中的
  $ docker ps 	
  # 查看所有
  $ docker ps -a
  ```

* 删除一个容器

  ```shell
  $ docker rm [OPTIONS] CONTAINER [CONTAINER...]
  # 强制删除
  $ docker rm -f name
  ```

* 容器启动/关闭和重启

  ```shell
  $ docker start/stop/restart Name
  ```

* 退出或关闭容器

  ```shell
  # 关闭并退出
  $ exit  ||  Ctrl + C
  # 退出但是不关闭
  # 快捷键: Ctrl + P + Q
  ```

* 连接到正在运行中的容器

  ```shell
  $ docker attach Name
  ```

* 在运行的容器中执行命令

  ```shell
  $ docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
  # 在容器 mynginx 中以交互模式执行容器内 /root/test.sh 脚本
  $ docker exec -it mynginx /bin/sh /root/test.sh
  ```

  

## 实例

### 部署Nginx并修改配置文件

```shell

docker run  --name myNginx -d -p 23380:80 -v /wwww/docker/myNginx/html:/usr/share/nginx/html -v /wwww/docker/myNginx/nginx.conf:/etc/nginx/nginx.conf:ro -v /wwww/docker/myNginx/conf.d:/etc/nginx/conf.d nginx
# 有瑕疵
```



[参考1](https://blog.csdn.net/wangfei0904306/article/details/77623400)

[参考2](https://blog.csdn.net/qq_26641781/article/details/80883192)