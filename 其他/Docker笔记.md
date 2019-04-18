#	Docker

------



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

