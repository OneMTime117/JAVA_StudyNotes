# Docker

## 1、什么是Docker

​		Docker是一个开源的应用容器引擎。可以让开发者打包自己的应用以及依赖包到一个轻量级

、可移植的容器中，然后可以发布到任何流行的Linux机器（也可以是windows）上，也可以实现虚拟化。

​		Docker类似于将已安装的软件编译成一个镜像，且镜像中以及做好了软件的配置。然后将镜像发布出去，可以被其他使用者直接使用这个镜像。镜像在使用时就称之为该软件的容器，容器启动时非常快的。

Docker对于开发者的作用：减少开发者在Linux上安装、配置大量软件的过程。

## 2、Docker核心概念

​	docker主机（Host）：安装了Docker程序的机器（安装在服务器上）

​	docker客户端（Client）：通过客服端来使用docker主机

​	docker仓库：存放docker镜像

​	docker镜像：软件打包好的镜像

​	docker容器：docker镜像启动后的实例称之为docker容器；容器时独立运行的一个应用

使用docker的步骤：

1、安装docker

2、去docker仓库找到队应软件docker镜像

3、使用docker运行镜像，生成一个docker容器

4、对容器停止，就是关闭对应软件

## 3、安装Docker

1、安装linux虚拟机

VMware（重量级虚拟机，要钱）、VirtualBox（开源，轻量）

2、安装linux系统

下载linux系统，如CentOS7

3、登入Linux系统   默认账号密码为 root/123456

一般情况下，不直接使用虚拟机的主机创建操作系统；而是使用SSH客户端进行连接

4、调试Linux系统网络

在虚拟机网络模块，设置网络连接方式为 桥接网卡，网卡选择当前物理机使用的，然后高级中选择接入网线

**并且将虚拟机中的网卡ip，配置为对应网关地址**

通过 **service network restart** 进网络重置

通过 **ip addr**  获取当前网络状态，此时就可以发现，默认的其中一个网卡信息中，ip地址与物理机地址处于同一个网段，说明网络设置成功

5、进行Docker安装

- 首先查看Centos版本，要求内核版本高于3.10  通过   **uname  -r** 进行查看
- 版本过低时，可以通过**yum  update**  进行升级
- 通过 yum install docker 进行docker安装
- 通过 **docker -v** 可以查看docker的版本号，可以检测docker是否成功安装
- 通过**systemctl start docker** 启动docker应用程序（可以通过 systemctl enable docker 设置程序开机自启动）
- 通过 systemctl stop docker 关闭docker应用程序

## 4、docker的使用

### 1、docker镜像操作（镜像获取部分）

- **docker search mysql**  docker镜像查询    在docker hub 网站中搜索mysql镜像，获取查询列表（包含mysql镜像名、 描述、标星次数、是否为官方镜像（一般官方镜像都为手动）、是否为自动配置）
- **docker pull mysql：tag**  docker镜像拉取（下载），tag为版本号，mysql为镜像名，只需要最底层路径名。如果不知道tag时，默认拉取latest 最新版本）
- 由于docker hub为外网地址，因此下载是需要设置docker进行加速器,对应CentOS7 可以通过修改 /etc/docker/daemon.json文件，加入{"registry-mirrors":["https://423s8foo.mirror.aliyuncs.com"]}（其中加速器地址可以通过阿里云免费获取）
- **docker images**  查看docker所有本地镜像
- **docker rmi  mysql**   删除指定本地镜像

### 2、docker 容器操作

- **docker run     --name 容器名  -d 镜像名：版本号 **     运行镜像，从而创建出一个容器（其中-d指后台运行,还可以添加 -p 参数，来指定主机端口与容器端口之间的映射 如   -p 3306：3306 ；由于容器中mysql设置的默认端口为3306，因此将主机端口 3306 映射到容器端口的3306上，此时访问主机端口3306时，就会映射访问到容器中的mysql  ）

- **docker ps**  查询系统目前有哪些docker容器在运行；（后缀， -a 可以查询所有容器状态，运行或没运行的所有容器）

- **docker stop  容器id/容器name**   停止容器运行（docker kill 容器id/容器name 为强制关闭容器）

- **docker start  容器id/容器name**   重启容器

- **docker rm   容器id/容器name**   删除容器（-f  可以删除运行中的容器）(docker container prune 删除所有未运行的容器 )

- **docker exec -it  容器id/容器name  bash**进入容器(使用exit离开)

- **docker  logs 容器id/容器name**  获取容器中，应用程序的运行日志

  **对于官方docker镜像。都有使用文档进行参考，获取更多的docker命令操作**

### 5、docker各种常用开发软件镜像启动规范：

- mysql： docker run --name mysql01  -e MYSQL_ROOT_PASSWORD=123456  -p 3306:3360 -d   -restart always  mysql:5.5

指定root用户登入的密码（否则mysql启动失败）、指定端口映射（用于访问mysql）

- redis :     docker run --name redis01    -p 6379:6379 -d   --restart always  redis

- zookeeper:  docker run --name zookeeper01 -p 2181:2181   -d  --restart always zookeeper

  其中--restart  always 指该容器伴随docker启动，而启动

# CentOS 操作：

yum（Yellow dogUpdate Modified），是存在于CentOS等Linux系统中的shell前端软件包管理器。基于RPM包（Linux软件包的一种文件格式）管理，能够从指定服务器上自动下载RPM包并按照，可以自动处理Linux软件安装中复杂依赖下载配置的操作，一次性安全所有依赖的软件包，无需重复下载、安装。

yum常用命令：

yum install xxx     安装软件   加上-y可以跳过是否安装的确认环节和是否需要安装某些非必须包的环节（全部默认为yes）

systemctl常用命令：

ps（process status 进程状态） 进程常用命令：

a：显示当前终端下的所有进程，可以为其他用户的进程（不同用户使用同一终端使用）

u：显示当前登入用户下的所有进程

-e 显示系统内的所有进程 

### 防火墙设置：

由于防护墙默认屏蔽所有端口的访问，因此需要进行开放端口设置，或者直接关闭防火墙

systemctl status firewalld.service    查看防火墙状态

systemctl stop firewalld.service   关闭防火墙

systemctl  disable firewalld.service   禁止防火墙开机自启动

firewall-cmd --list-ports   查看防火墙中已开放的端口

firewall-cmd --zone=public --add-port=2181/tcp --permanent    开放指定端口（需要重启防火墙，才能真正开放）

firewall-cmd --zone=public --remove-port=3306/tcp --permanent  关闭指定端口（需要重启防火墙，才能真正关闭）

firewall-cmd --reload     重启防火墙

